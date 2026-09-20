1. Comparison of All 6 Programs

Exercise	Collectives Used	     Memory Allocation	                                         Manual Summation on Root?	Final Result Available
Exercise 1	MPI_Bcast	             Every process allocates the full array	                   Yes (MPI_Send/MPI_Recv)	Root only
Exercise 2	MPI_Scatter	             Root allocates full array, others allocate only a chunk	   Yes (MPI_Send/MPI_Recv)	Root only
Exercise 3	MPI_Scatter, MPI_Gather	     Root allocates full array, others allocate only a chunk	   Yes (sum gathered array)	Root only
Exercise 4	MPI_Scatter, MPI_Reduce	     Root allocates full array, others allocate only a chunk	    No	                        Root only
Exercise 5 	MPI_Scatter, MPI_Allreduce   Root allocates full array, others allocate only a chunk	    No	                        All processes
Exercise 6	MPI_Scatter, MPI_Scan	     Root allocates full array, others allocate only a chunk	    No	                        Different result per rank (prefix sums)





2. Run all 6 programs with 2, 4, and 8 processes

Exercise	          2 Processes	4 Processes	8 Processes
Broadcast + Send/Recv	  0.0387 s	0.0410 s	0.0343 s
Scatter + Send/Recv	  0.0228 s	0.0188 s	0.0117 s
Scatter + Gather	  0.0210 s	0.0183 s	0.0114 s
Scatter + Reduce	  0.0214 s	0.0241 s	0.0196 s
Scatter + Allreduce	  0.0213 s	0.0201 s	0.0176 s
Scatter + Scan	          0.0192 s	0.0172 s	0.0092 s


Fastest Approach
With 2 Processes => MPI_Scan = 0.0192 sec
With 4 Processes => MPI_Scan = 0.0172 sec
With 8 Processes => MPI_Scan = 0.0092 sec
Overall Fastest  => Exercise 6 (MPI_Scan) with 0.0092 seconds using 8 processes.

Why?
MPI_Scan achieves higher performance and scalability at scale because it divides work across processes and uses logarithmic, tree-based communication algorithms rather than linear point-to-point transfers (MPI_Send/MPI_Recv). By computing prefix reductions in parallel along a communication tree, it minimizes latency and message overhead compared to broadcasting large datasets (MPI_Bcast), which redundantly copies full data arrays to every rank, or scattering full arrays (MPI_Scatter), which still requires additional gather and coordination steps to calculate cumulative offsets.






3. Thinking question: In what situation would you choose MPI_Scan over MPI_Allreduce? Give a concrete example.

MPI_Scan is selected over MPI_Allreduce when processes require individual cumulative results (prefix sums) based on their rank order rather than a single, uniform global value. While MPI_Allreduce combines data across all ranks and distributes the same total to every process, MPI_Scan computes a running partial reduction up to each process's rank. This makes MPI_Scan ideal for scenarios where operations depend on sequence or position across the distributed communicator.

A common application of MPI_Scan is calculating dynamic file offsets or array indices during parallel write operations. For instance, if four processes generate 100, 150, 200, and 250 records respectively, MPI_Scan allows each rank to determine its precise starting location in a shared global output file (offsets 0, 100, 250, and 450) without overlap or extra communication overhead. In contrast, MPI_Allreduce would only provide the total count of 700 records to all processes, leaving them without the intermediate prefix information necessary to coordinate parallel file access, load balancing, or distributed indexing.