More complex cases can be facilitated with an immediately-invoked lambda.
// Bad Idea
std::string somevalue;
if (caseA) {
somevalue = "Value A";
} else if(caseB) {
somevalue = "Value B";
} else {
somevalue = "Value C";
}
// Better Idea
const std::string somevalue = [&](){
if (caseA) {
return "Value A";
} else if (caseB) {
return "Value B";
} else {
return "Value C";
}
}();

https://articles.emptycrate.com/2014/12/16/complex_object_initialization_optimization_with_iife_in_c11.html