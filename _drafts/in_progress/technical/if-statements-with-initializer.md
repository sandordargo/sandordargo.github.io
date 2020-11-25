Note: In C++17, if statements can contain an initializer, so the scope of the declaration can be limited while avoiding the implicit conversion:


//   std::cout << " What does an invalid bucketIdx have?\n";
//   std::cout << " bucket #" << (nbuckets) << " has " << mymap.bucket_size(nbuckets) << " elements.\n";