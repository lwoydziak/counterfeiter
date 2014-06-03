Counterfeiter
=============

[![Build Status](https://travis-ci.org/maxbrunsfeld/counterfeiter.svg?branch=master)](https://travis-ci.org/maxbrunsfeld/counterfeiter)

When writing unit-tests for an object, it is often useful to have fake implementations
of the object's collaborators. In go, such fake implementations cannot be generated
automatically at runtime. This tool allows you to generate them before compilation.

### Generating fakes

Choose an interface for which you would like a fake implementation:

```shell
$ cat path/to/some_package/something.go
```

```go
package some_package

type Something interface {
	DoThings(string, uint64) (int, error)
	DoNothing()
}
```

Run counterfeiter like this:

```shell
$ counterfeiter path/to/some_package Something
```

```
Wrote `FakeSomething` to `path/to/some_package/fakes/fake_something.go`
```

You can customize the location of the ouptut using the `-o` flag, or write the code to standard out by providing `-` as a third argument.

### Using the fake in your tests

Instantiate fakes with `new`:

```go
import "my-repo/path/to/some_package/fakes"

var fake = new(fakes.FakeSomething)
```

Fakes record the arguments they were called with:

```go
fake.DoThings("stuff", 5)

Expect(fake.DoThingsCallCount()).To(Equal(1))

str, num := fake.DoThingsArgsForCall(0)
Expect(str).To(Equal("stuff"))
Expect(num).To(Equal(uint64(5)))
```

You can set their return values:

```go
fake.DoThingsReturns(3, errors.New("the-error"))

num, err := fake.DoThings("stuff", 5)
Expect(num).To(Equal(3))
Expect(err).To(Equal(errors.New("the-error")))
```

You can also supply them with stub functions:

```go
fake.DoThingsStub = func(arg1 string, arg2 uint64) (int, error) {
	Expect(arg1).To(Equal("stuff"))
	Expect(arg2).To(Equal(uint64(5)))
	return 3, errors.New("the-error")
}

num, err := fake.DoThings("stuff", 5)

Expect(num).To(Equal(3))
Expect(err).To(Equal(errors.New("the-error")))
```

### License

Counterfeiter is MIT-licensed.
