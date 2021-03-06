# CS 6210: Distributed Service using grpc - Project 3

Submitted by : Jyoti Saini & Gauri Joshi


INTRODUCTION:

In this project, we implement a simple distributed service using grpc. We build a store which receives requests from different users, querying the prices offered by the different registered vendor. The store is provided with a file of ip address:port of vendor servers. On each product query, your server sends a request all of these vendor servers for their bid on the queried product. Store then receives responses from all the vendors, collates the (bid, vendor_id) from the vendors and sends it back to the requesting client

*****************************************ThreadPool Implementation*****************************************

We maintain a queue of concurrent client requests in the queue mJobs and a mutex to acquire lock over the queue whenever a job is added or removed from the queue. On server run, we spawn numThreads threads which keeps an eye on the the mJobs Queue. Whenever a client requests comes to store, we push the task in to the mJobs queue. All Worker threads are looping over the mJobs queue if there are any job to be performed. All the requests are pushed in the queue and are picked uo by the worker threads. Async API completetionqueue->next() is Thread Safe .

Upon receiving a client request, our store assigns a thread from the thread pool to the incoming request for processing.
- The thread makes async RPC calls to the vendors
- The thread awaits for all results to come back
- The thread then collates the results
- The thread replies to the store client with the results of the call
- Having completed the work, the thread then return to the thread pool.


**************************************Store Implementation******************************************

- The store created for the key value store implementation, performs the functionality of a server ( serves the requests generated by the client)as well as a client by requesting the bids for various products to the vendor. 
- The vendors are basically servers listening on different ports. On running the store, it starts its service on a port number provided by the user. The port used for the purpose of this project is 50061. 
- After the store gets the addresses of the vendor servers from a file it stores them in a vector. Then the function HandleRpcs is called which in turn creates a new instance of the CallData class. 
- Inside this class, the store program then waits for the client request. Once the client request is made, the store loops over the vendor addresses and calls the VendorClient function to get the bids for the products as requested by the client. 
- The vendorclient method returns the bids of the products to the store. 
- The calls made from client to the store as well as those made from the store to the Vendor are asynchronous. A single completion queue is maintained for the incoming client requests and for the server-vendor communication, each thread has its own completion queue. 

*********************************************TO RUN*********************************************

- Goto src directory and run `make` command
  - Two binaries will be generated  Store and threadpool. threadpool binary is used within store.cc.
  - run command ./store $server_port $file_path_for_vendor_addrs $numThreads. This will bring the server up and create threadpool with the number of threads passed as commandline.
  - Our store starts listening on a port(for clients to connect to) given as command line argument.

- Goto test directory and run `make` command
  - Two binaries would be created `run_tests` and `run_vendors`
  - First run the command `./run_vendors ../src/vendor_addresses.txt &` to start a process which will run multiple servers on different threads listening to (ip_address:ports) from the file given as command line argument.
  - Then finally run the command `./run_tests  $port_on_which_store_is_listening  $max_num_concurrent_client_requests` to start a process which will simulate real world clients sending requests at the same time.
  - This process read the queries from the file `product_query_list.txt`
 
