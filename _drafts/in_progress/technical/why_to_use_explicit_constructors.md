Explicit constructors

What are explicit constructors and what are its advantages?

The `explicit` specifier specifies that a constructor cannot be used for implicit conversions.

If not used, the compiler is allowed to make one implicit conversion to resolve the parameters to a function. The compiler can use constructors callable with a single parameter to convert from one type to another in order to get the right type for a parameter.

Let's take this example:


class WrappedInt {
public:
  // single parameter constructor, can be used as an implicit conversion
  WrappedInt(int number) : m_number (number) {}

  // using explicit, implicit conversions are not allowed
  // explicit WrappedInt(int number) : m_number (number) {}

  int GetNumber() { return m_number; }

private:
  int m_number;
};

void doSomething(WrappedInt aWrappedInt) {
  int i = aWrappedInt.GetNumber();
}

int main() {
  doSomething(42);
}

The argument is not a `WrappedInt` object, just a plain old `int`. However, there exists a constructor for `WrappedInt` that takes an `int` so this constructor can be used to convert the parameter to the correct type.

The compiler is allowed to do this once for each parameter.

Prefixing the explicit keyword to the constructor prevents the compiler from using that constructor for implicit conversions. Adding it to the above class will create a compiler error at the function call `doSomething(42)`. It is now necessary to call for conversion explicitly with `doSomething(WrappedInt(42))`.

The reason you might want to do this is to avoid accidental construction that can hide bugs. Let's think about strings. You have a `MySpecialString` class with a constructor taking an `int`. This constructor creates a string with the size of the passed in `int`. You have a function `print(const MySpecialString&)`, and you call print(3) (when you actually intended to call print("3")). You expect it to print "3", but it prints an empty string of length 3 instead.

A third example, is also about strings, let's say this is `MySpecialString`:

class MySpecialString {
public:
    MySpecialString(int n); // allocate n bytes to the MySpecialString object
    MySpecialString(const char *p); // initializes object with char *p
};

What happens with this constructor call?

MySpecialString mystring = 'x';

The character 'x' will be implicitly converted to int and then the MySpecialString(int) constructor will be called. But, this is not what the user might have intended. So, to prevent such conditions, we shall define the constructor as explicit:

class MySpecialString {
public:
    explicit MySpecialString (int n); //allocate n bytes
    MySpecialString(const char *p); // initialize sobject with string p
};

To avoid such subtle bugs, one should always consider making single argument constructors explicit initially (more or less automatically), and removing the explicit keyword only when the implicit conversion is wanted by design.