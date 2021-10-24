---
title: rgat
subtitle: Development - Instrumentation
layout: page
menubar: devdocs_menu
email: niacatn@gmail.com
hero_height: is-fullwidth
hero_darken: true

---
# Instrumentation


rgat currently uses a Pin tool for gathering instruction traces and API call data

As discussed on the [Libraries](/devdocs/libraries) page - earlier versions of rgat used DynamoRIO, and rgat may switch back to or support that in parallel.

### Code Instrumentation

The instrumentation tool communicates to rgat over 4 different type of named pipe (on Windows, the Linux method of IPC is undecided)
* Basic Block
    - Workers/Pipes: One per traced process
    - Content: Blobs of instruction opcodes and addresses
    - When: As Pin encounters them
* Trace Tag
    - Workers/Pipes: One per traced thread
    - Encoded descriptions of which basic blocks are executed 
    - When: Each time a block executes

### Event Instrumentation

A third worker - (called the Module worker) has two responsibilities
  * Receives event information such as threads starting/stopping or log entry emission
  * Receives API tracing data such as parameters and return values, which has to be reconciled with the basic blocks which caused them to be generated
  * Sends user commands and settings back to the traced process. This can include module trace/ignore lists and debug commands.

### API Instrumentation

This is a difficult area which requires significant work - currently rgat requires a description of individual API calls to be baked into the Pin tool. Here is example code for handling of a single Windows API call - Advapi32::RegCreateKeyExA:

```cpp
	WINDOWS::LSTATUS replacement_RegCreateKeyExA(
		LEVEL_PINCLIENT::CONTEXT* ctx, THREADID threadid, UINT32 tlskey, AFUNPTR funcaddr,
		WINDOWS::HKEY  hKey,
		WINDOWS::LPCSTR lpSubKey,
		WINDOWS::DWORD Reserved,
		WINDOWS::LPSTR lpClass,
		WINDOWS::DWORD dwOptions,
		WINDOWS::REGSAM samDesired,
		const WINDOWS::LPSECURITY_ATTRIBUTES lpSecurityAttributes,
		WINDOWS::PHKEY phkResult,
		WINDOWS::LPDWORD lpdwDisposition)
	{

		threadObject* threaddata = static_cast<threadObject*>(PIN_GetThreadData(tlskey, threadid));

		if (threaddata->lastBlock->blockID != -1)
		{
			std::string key = HKEY_to_string(hKey);

			if (!key.empty())
				fprintf(threaddata->threadpipeFILE, ARG_MARKER",%d," PTR_prefix ",%lx,M,%s\x01", 0, (void*)funcaddr, (void*)threaddata->lastBlock->blockID, key.c_str());
			else
				fprintf(threaddata->threadpipeFILE, ARG_MARKER",%d," PTR_prefix ",%lx,M,0x%lx\x01", 0, (void*)funcaddr, (void*)threaddata->lastBlock->blockID, hKey);
			
			if (lpSubKey != 0)
			{
				fprintf(threaddata->threadpipeFILE, ARG_MARKER",%d," PTR_prefix ",%lx,M,%s\x01", 1, (void*)funcaddr, (void*)threaddata->lastBlock->blockID, lpSubKey);
			}
			else
			{
				fprintf(threaddata->threadpipeFILE, ARG_MARKER",%d," PTR_prefix ",%lx,M,NULL\x01", 1, (void*)funcaddr, (void*)threaddata->lastBlock->blockID);
			}
		}

		WINDOWS::LSTATUS retval;
		PIN_CallApplicationFunction(ctx, threadid, CALLINGSTD_STDCALL, funcaddr, NULL,
			PIN_PARG(WINDOWS::LSTATUS), &retval,
			PIN_PARG(WINDOWS::HKEY), hKey,
			PIN_PARG(WINDOWS::LPCTSTR), lpSubKey,
			PIN_PARG(WINDOWS::DWORD), Reserved,
			PIN_PARG(WINDOWS::LPWSTR), lpClass,
			PIN_PARG(WINDOWS::DWORD), dwOptions,
			PIN_PARG(WINDOWS::REGSAM), samDesired,
			PIN_PARG(WINDOWS::LPSECURITY_ATTRIBUTES), lpSecurityAttributes,
			PIN_PARG(WINDOWS::PHKEY), phkResult,
			PIN_PARG(WINDOWS::LPDWORD), lpdwDisposition,
			PIN_PARG_END());

		if (threaddata->lastBlock->blockID != -1)
		{
			fprintf(threaddata->threadpipeFILE, ARG_MARKER",%d," PTR_prefix ",%lx,M,0x%lx\x01", 7, (void*)funcaddr, (void*)threaddata->lastBlock->blockID, *phkResult);
			fprintf(threaddata->threadpipeFILE, RETVAL_MARKER"," PTR_prefix ",%lx,%s\x01", (void*)funcaddr, (void*)threaddata->lastBlock->blockID, ErrorCodeToString(retval).c_str());
			fflush(threaddata->threadpipeFILE);
		}
		return retval;
	}
```
The whole function prototype has to be replicated and then different possible combination of parameters have to be handled and it's a horrendous amount of work considering how many APIs there are.

In addition to this, rgat itself also needs a JSON API details entry provided, so that instrumentation data can be interpreted on the analysis chart. The API tracking has the concept of entities (things, like files, registry keys, network addresses, etc) and references which refer to entities (HANDLES, sockets, etc).

```json
             "RegCreateKeyExA": { 
				"Filter" : "Registry",
				"Label": "Create",
				"Effects": [{ "Type": "LinkReference", "EntityIndex": 1, "ReferenceIndex": 7 }],
				"Parameters": [
					{"Index":-1, "Name":"Result", "Type":"Info"},
					{"Index":0, "Name":"hKey", "Type":"EntityModifier"},
					{"Index":1, "Name":"lpSubKey", "Type":"Entity", "EntityType": "Registry", "RawType": "Path"},
					{"Index":7, "Name":"phkResult", "Type":"Reference", "EntityType": "Registry", "RawType": "HKEY", "Conditional":true}
				]
			 },
```

The above entry causes a call to RegCreateKeyExA to link the *HKEY* **reference** in parameter 7 to the *lpSubKey* **entity** in parameter 1. There is also the concept of modifiers (like the parent hKey in paramter 0) and other bits of information about the operation such as the result in the return value (param -1).

Implementing all this has made it possible to get the analysis chart working in its current state with a handful of APIs, but I'm looking for a better way of doing it before putting more APIs in there - particularly a way of doing it all in one file, perhaps like how API monitor does it. It's likely this will mean having to manually instrument API calls though rather than relying on PIN_CallApplicationFunction.

## Performance Considerations

*Background to how instruction traces are gathered performantly is covered in the relevant [Blog post](/ff)*

[Intel has some tips on instrumentation performance](https://software.intel.com/sites/landingpage/pintool/docs/98437/Pin/html/index.html#PERFORMANCE), some of which rgat probably needs to follow more closely.

The work of the Pin tool is split into two stages: *Analysis* and *Instrumentation*

#### Analysis 

Performed by InstrumentNewTrace in pingat.cpp, this 

* Decides if a block should be instrumented, based on the users trace/ignore lists
    - Future enhancement: Decide based on addresses to limit instrumentation to parts of a binary
* If no - analysis ends
* Sends the instruction data to rgat on the Basic Block pipe
* Builds a basic block metadata object which will be passed to the later instrumentation passes
* Inserts instrumentation code at the end of the basic block, specific to the type of control flow being performed

Analysis is performed relatively rarely, so its fine to do a relatively large amount of processing here to save even a small amount of processing in the instrumentation stage

#### Instrumentation

After a block actually executes (**every time** it executes), control flow passes to one of these instrumentation functions that we added to the block in the analysis stage. 

```cpp
VOID at_unconditional_branch(BLOCKDATA* block_data, ADDRINT targetBlockAddress, THREADID threadid)
{
	threadObject* thread = static_cast<threadObject*>(PIN_GetThreadData(tls_key, threadid));
	RecordEdge(thread, block_data, targetBlockAddress);
}

VOID at_conditional_branch(BLOCKDATA* block_data, bool taken, 
  ADDRINT targetBlockAddress, ADDRINT fallthroughAddress, THREADID threadid)
{
	threadObject* thread = static_cast<threadObject*>(PIN_GetThreadData(tls_key, threadid));
	RecordEdge(thread, block_data, taken ? targetBlockAddress : fallthroughAddress);
}
```

The performance of rgat hinges massively on RecordEdge, which has three parts

1. Records the execution of the block
2. Manages the activity level of the basic block and, based on this, the thread
3. Either record what happened (fast) or report what happened (slow)

This sounds like a lot of stuff to do on **every single basic block execution** but the workload is very front loaded to happen mostly only when control flow changes. This means that in regions of code with millions/billions of iterations there is very little overhead or data transfer, which compares to standard tracing schemes where busy code produces gigabytes of data with a terrible performance impact.

## Future work

#### Dealing with high complexity, low iteration code

  In regions of code which don't 'spin' very much the performance suffers as there is less incrementing of counters and more sending of trace data. I find that this type of code tends to block a lot more, which gives rgat time to catch up. 
  A future optimisation may be to switch between different instrumentation strategies as the type of code being traced changes.  

#### Finer grained instrumentation choices
  
  Ultimately, computers will always be able to generate traces faster than they can record them, put them on a graph and plot them out in a visualisation. The solution to this will probably be to offer low-overhead, low-detail tracing modes to allow the analyst to choose which areas of code to subject to more detailed instrumentation. 

#### Branch Tracing
  
  Recent Intel processors have a [Processor Trace](https://software.intel.com/content/www/us/en/develop/blogs/processor-tracing.html) feature which allows the gathering of trace data to memory with a 15%~ overhead. This is probably significantly faster than the instrumentation model being used here, with some caveats:
  * Restricts rgat to Intel processors, unless AMD comes up with something similar (have they? will they?)
  * Instrumentation will likely still be needed for fine-grained control
  * I haven't tested it and trace rates are currently not enough of a bottleneck for a potential optimisation to be a development priority right now (with visualisation quality, trace accuracy and analysis features all competing for time)

So this will likely remain on the TODO list until trace gathering speed is the big bad problem holding rgat back.
  
### Related Trello Tickets

[Gather trace through Intel Processor Trace](https://trello.com/c/QiQlcD0f/179-gather-trace-through-intel-processor-trace)

[DynamoRIO for tracing](https://trello.com/c/jvbmuXLs/180-dynamorio-for-tracing)