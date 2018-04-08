ignatchenko's slide
don't do from main RAM read to down to thread context switch

L1 - separate instruction and data cache. why? you don't want to fill everything with data, instructions would be wiped out.
L2-L3 - doesn't have it. inclusive caches, what is L1 is in L2... instructions are RO (almost always, no need to write back)

L1 vs RAM two orders of magnitude (100x)

speed of light issues

mem is the bottleneck, not the CPU. number of cachelines per second is critical