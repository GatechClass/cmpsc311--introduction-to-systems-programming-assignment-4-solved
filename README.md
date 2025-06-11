# cmpsc311--introduction-to-systems-programming-assignment-4-solved
**TO GET THIS SOLUTION VISIT:** [CMPSC311- Introduction to Systems Programming Assignment 4 Solved](https://mantutor.com/product/cmpsc311-introduction-to-systems-programming-fall-assignment-4-solved/)


---

**For Custom/Order Solutions:** **Email:** mantutorcodes@gmail.com  

*We deliver quick, professional, and affordable assignment help.*

---

<h2>Description</h2>



<div class="kk-star-ratings kksr-auto kksr-align-center kksr-valign-top" data-payload="{&quot;align&quot;:&quot;center&quot;,&quot;id&quot;:&quot;66857&quot;,&quot;slug&quot;:&quot;default&quot;,&quot;valign&quot;:&quot;top&quot;,&quot;ignore&quot;:&quot;&quot;,&quot;reference&quot;:&quot;auto&quot;,&quot;class&quot;:&quot;&quot;,&quot;count&quot;:&quot;0&quot;,&quot;legendonly&quot;:&quot;&quot;,&quot;readonly&quot;:&quot;&quot;,&quot;score&quot;:&quot;0&quot;,&quot;starsonly&quot;:&quot;&quot;,&quot;best&quot;:&quot;5&quot;,&quot;gap&quot;:&quot;4&quot;,&quot;greet&quot;:&quot;Rate this product&quot;,&quot;legend&quot;:&quot;0\/5 - (0 votes)&quot;,&quot;size&quot;:&quot;24&quot;,&quot;title&quot;:&quot;CMPSC311- Introduction to Systems Programming Assignment 4 Solved&quot;,&quot;width&quot;:&quot;0&quot;,&quot;_legend&quot;:&quot;{score}\/{best} - ({count} {votes})&quot;,&quot;font_factor&quot;:&quot;1.25&quot;}">

<div class="kksr-stars">

<div class="kksr-stars-inactive">
            <div class="kksr-star" data-star="1" style="padding-right: 4px">


<div class="kksr-icon" style="width: 24px; height: 24px;"></div>
        </div>
            <div class="kksr-star" data-star="2" style="padding-right: 4px">


<div class="kksr-icon" style="width: 24px; height: 24px;"></div>
        </div>
            <div class="kksr-star" data-star="3" style="padding-right: 4px">


<div class="kksr-icon" style="width: 24px; height: 24px;"></div>
        </div>
            <div class="kksr-star" data-star="4" style="padding-right: 4px">


<div class="kksr-icon" style="width: 24px; height: 24px;"></div>
        </div>
            <div class="kksr-star" data-star="5" style="padding-right: 4px">


<div class="kksr-icon" style="width: 24px; height: 24px;"></div>
        </div>
    </div>

<div class="kksr-stars-active" style="width: 0px;">
            <div class="kksr-star" style="padding-right: 4px">


<div class="kksr-icon" style="width: 24px; height: 24px;"></div>
        </div>
            <div class="kksr-star" style="padding-right: 4px">


<div class="kksr-icon" style="width: 24px; height: 24px;"></div>
        </div>
            <div class="kksr-star" style="padding-right: 4px">


<div class="kksr-icon" style="width: 24px; height: 24px;"></div>
        </div>
            <div class="kksr-star" style="padding-right: 4px">


<div class="kksr-icon" style="width: 24px; height: 24px;"></div>
        </div>
            <div class="kksr-star" style="padding-right: 4px">


<div class="kksr-icon" style="width: 24px; height: 24px;"></div>
        </div>
    </div>
</div>


<div class="kksr-legend" style="font-size: 19.2px;">
            <span class="kksr-muted">Rate this product</span>
    </div>
    </div>
<h1>Overview</h1>
In general, caches store <em>key </em>and <em>value </em>pairs in a fast storage medium. For example, in a CPU cache, the key is the memory address, and the value is the data that lives at that address. When the CPU wants to access data at some memory address, it first checks to see if that address appears as a key in the cache; if it does, the CPU reads the corresponding data from the cache directly, without going to memory because reading data from memory is slow.

In a browser cache, the key is the URL of an image, and the value is the image file. When you visit a web site, the browser fetches the HTML file from the web server, parses the HTML file and finds the URLs for the images appearing on the web page. Before making another trip to retrieve the images from the web server, it first checks its cache to see if the URL appears as a key in the cache, and if it does, the browser reads the image from local disk, which is much faster than reading it over the network from a web server.

In this assignment you will implement a block cache for mdadm. In the case of mdadm, the key will be the tuple consisting of disk number and block number that identifies a specific block in JBOD, and the value will be the contents of the block. When the users of mdadm system issue mdadm_read call, your implementation of mdadm_read will first look if the block corresponding to the address specified by the user is in the cache, and if it is, then the block will be copied from the cache without issuing a slow JBOD_READ_BLOCK call to JBOD. If the block is not in the cache, then you will read it from JBOD and insert it to the cache, so that if a user asks for the block again, you can serve it faster from the cache.

<h1>Cache Implementation</h1>
Typically, a cache is an integral part of a storage system and it is not accessible to the users of the storage system. However, to make the testing easy, in this assignment we are going to implement cache as a separate module, and then integrate it to mdadm_read and mdadm_write calls.

Please take a look at cache.h file. Each entry in your cache is the following struct.

typedef struct { bool valid; int disk_num; int block_num; uint8_t block[JBOD_BLOCK_SIZE]; int access_time;

} cache_entry_t;

The valid field indicates whether the cache entry is valid. The disk_num and block_num fields identify the block that this cache entry is holding and the block field holds the data for the corresponding block. The access_time field stores when the cache element was last accessed—either written or read.

The file cache.c contains the following predefined variables.

static cache_entry_t *cache = NULL; static int cache_size = 0; static int clock = 0; static int num_queries = 0; static int num_hits = 0;

Now let’s go over the functions declared in cache.h that you will implement and describe how the above variables relate to these functions. You must look at cache.h for more information about each function.

<ol>
<li>int cache_create(int num_entries); Dynamically allocate space for num_entries cache entries and should store the address in the cache global variable. It should also set cache_size to num_entries, since that describes the size of the cache and will also be used by other functions. Calling this function twice without an intervening cache_destroy call (see below) should fail. The num_entries argument can be 2 at minimum and 4096 at maximum.</li>
<li>int cache_destroy(void); Free the dynamically allocated space for cache, and should set cache to NULL, and cache_size to zero. Calling this function twice without an intervening cache_create call should fail.</li>
<li>int cache_lookup(int disk_num, int block_num, uint8_t *buf); Lookup the block identified by disk_num and block_num in the cache. If found, copy the block into buf, which cannot be NULL. This function must increment num_queries global variable every time it performs a lookup. If the lookup is successful, this function should also increment num_hits global variable; it should also increment clock variable and assign it to the access_time field of the corresponding entry, to indicate that the entry was used recently. We are going to use num_queries and num_hits variables to compute your cache’s hit ratio.</li>
<li>int cache_insert(int disk_num, int block_num, uint8_t *buf); Insert the block identified by disk_num and block_num into the cache and copy buf—which cannot be NULL—to the corresponding cache entry. Insertion should never fail: if the cache is full, then an entry should be overwritten according to the LRU policy using data from this insert operation. This function should also increment and assign clock variable to the access_time of the newly inserted entry.</li>
<li>void cache_update(int disk_num, int block_num, const uint8_t *buf); If the entry exists in cache, updates its block content with the new data in buf. Should also update the access_time if successful.</li>
<li>bool cache_enabled(void); Returns true if cache is enabled. This will be useful when integrating the cache to your mdadm_read and mdadm_write functions.</li>
</ol>
<h1>Strategy for Implementation</h1>
The tester now includes new tests for your cache implementation. You should first aim to implement functions in cache.c and pass all the tester unit tests. Once you pass the tests, you should incorporate your cache into your mdadm_read and mdadm_write functions—you need to implement caching in mdadm_write as well, because we are going to use write-through caching policy, as described in the class. Once you do that, make sure that you still pass all the tests.

Next, try your implementation on the trace files and see if it improves the performance. To evaluate the performance, we have introduced a new cost is a metric into JBOD for measuring the effectiveness of your cache, which is calculated based on the number of operations executed. Each JBOD operation has a different cost, and by effective caching, you reduce the number of read operations, thereby reducing your cost. Now, the tester also takes a cache size when used with a workload file, and prints the cost and hit rate at the end. The cost is computed internally by JBOD, whereas the hit rate is printed by cache_print_hit_rate function in cache.c. The value it prints is based on num_queries and num_hits variables that you should increment.

Here’s how the results look like with the reference implementation. First, we run the tester on random input file:

$ ./tester -w traces/random-input &gt;x

Cost: 18948700

Hit rate: -nan%

The cost is 18948700, and the hit rate is undefined because we have not enabled cache. Next, we rerun the tester and specify a cache size of 1024 entries, using -s option:

$ ./tester -w traces/random-input -s 1024 &gt;x

Cost: 17669400

Hit rate: 24.5%

As you can see, the cache is working, given that we have non-zero hit rate, and as a result, the cost is now reduced. Let’s try it one more time with the maximum cache size:

$ ./tester -w traces/random-input -s 4096 &gt;x

Cost: 13091800

Hit rate: 87.9%

$ diff x traces/random-expected-output

$

Once again, we significantly reduced the cost using a larger cache. We also make sure that introducing caching does not violate correctness by comparing the outputs. If introducing a cache violates correctness of your mdadm implementation, you will get a zero grade for the corresponding trace file.
