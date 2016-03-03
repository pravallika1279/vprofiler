# VProfiler

VProfiler is a profiling tool that can efficiently and rigorously decompose the variance of the overall execution time of an application by automatically instrumenting its source code, and identifying the major functions that contribute the most to the overall variance. This tool is still at an early version, and it supports only MySQL (C++) and Postgres (C). Depending on the exact setting of the software system, a scrpt can be used to connect the four parts of this tool to make it fully automatic.

### Compilation:
Run script `compile.sh` to compile the code.
```
chmod +x compile.sh
./compile.sh
```

This tool contains four parts:

1. ExecutionTimeTracer: the tracer collects execution time data for a certain function and all the function called in that function. It also collects latency data. Appropriate functions should be called in order to mark the start and end of a query and transaction. Data files will be generated after each run with all the execution time data for further analysis. The current version of this tool supports only MySQL and Postgres. 

   Usage: Add the files to the code base of the application so that the defined functions can be called.
   		  In MySQL, call function `TraceTool::get_instance()->start_new_query()`, `TraceTool::get_instance()->query_end()` and `TraceTool::get_instance()->end_transaction()` at the appropriate location to mark the start, end of a query and end of a transaction. Set `TraceTool::is_commit` to true if the query is a commit query.
   		  In Postgres, call function `TRX_START()`, `TRX_END()` at the appropriate location to mark the start and end of a transaction. Call `COMMIT()` if the query is a commit query.
   		  The data files will be put in folder `latency`. The exact location of this folder depends on the software system being profiled. Make sure that this folder exists before running experiments.

2. SourceAnnotator: the annotator instruments the source code of the software application. It inserts appropriate function calls to call the functions defined in ExectuionTimeTracer to mark the start and end of a function call.

   Usage:
   ```
   ./annotate.sh <source code file> <function name> <location oftrace_tool.cc file>
   ```

   This tool updates file `trace_tool.cc`. Use it for further experiments.

3. VarianceBreaker: Given the execution time data generated by ExecutionTimeTracer, the breaker breaks down the variance of a function into variance and covarainces of its child functions and add corresponding entries to the variance tree file contained in the FactorSelector.

   Usage:
   ```
   Usage: var_breaker.py -f <function names file generated from SourceAnnotator>
                         -d <dir of data files generated from ExectuionTimeTracer>
                         -v <variance tree file>
                         -h print help message
   ```

4. FactorSelector: Given the variance tree generated by the VarianceBreaker, the selector performs a selections on the variance tree to select the top k most interesting functions. Note that this tool requires a pre-generated static call graph for the software application. Such call graph can be generated using tools like [CodeViz](http://www.csn.ul.ie/~mel/projects/codeviz/). A file containing the heights of each function in the call graph needs to be generated, each line in the form `func_name,height`.

   Usage:
   ```
   java FactorSelector <variance tree file> <heights file> <k(number of functions to select)> <root function> [selected function 1] [selected function 2]...
   ```
