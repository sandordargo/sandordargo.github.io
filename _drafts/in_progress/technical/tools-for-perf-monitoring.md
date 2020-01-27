c++ sanitizer to check for leaks. faster than valgrind

using godbolt compiler explorer

if you work on a small data set and you know it'll be small, don't care about big O...




typo valgrind --tool=cachegrind --branch-sim=yes –cache-sim=yes \
--cachegrind-out-file=chg.out ./myprog missing dash in cache-sim
