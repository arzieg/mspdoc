# Test

## file structure

* Test files end with _test.go and have TestXXX functions
* can be in package dir or in separte dir
* tests are not run if the source was not changed since the last test
* focus unit, integration testing, white-black box testing

```bash
# Alle Tests
go test ./...

# ausgeählte Tests
go test -v -run="French|Canal"
```

**special test-only packages**

* if you need to add test-only code as part of a package, you can place it in a package that end in _test
* That package, like XXX_test.go files will not be included in a regular build
* unlike normal test files, if will only be allowed to access exported identifiers, so it's usefule for "opaque" or "black-box" tests

```go
package myfunc_test
// this package is not part of package myfunc, so it has no internal access
```

## function structure

* Test functions have the same signature using testing.T
* errors are reported through parameter t and fail the test


**Standard**

```
func TestCrypto(t *testing.T) {
    ...
}
```

**Table-driven tests (set of values)**

```go
func TestValueFreeFloat(t *testing.T){
    table := []struct {
        v float64
        s string
    }{
        {1, "1"},
        {1.1, "1.1"},
    }

    for _, tt := range table {
        v := Value{T: floater, V: tt.v, m: &Machine{}}
        if s := v.String(); s!= tt.s {
            t.Errorf("%: wanted %s, got %s", tt.v, tt.s,s)
        }
    }
}

// Anderes Beispiel:
func TestIsPalindrome(t *testing.T) {
	var tests = []struct {
		input string
		want  bool
	}{
		{"", true},
		{"a", true},
		{"aa", true},
		{"ab", false},
		{"kayak", true},
		{"detartrated", true},
		{"A man, a plan, a canal: Panama", true},
		{"Evil I did dwell; lewd did I live.", true},
		{"Able was I ere I saw Elba", true},
		{"été", true},
		{"Et se resservir, ivresse reste.", true},
		{"palindrome", false}, // non-palindrome
		{"desserts", false},   // semi-palindrome
	}
	for _, test := range tests {
		if got := IsPalindrome(test.input); got != test.want {
			t.Errorf("IsPalindrome(%q) = %v", test.input, got)
		}
	}
}

```

**Table-driven subtests**

* Präferiert, wenn man größere Testcases hat, denn dann sieht man, an welchem Subtask das Problem aufgetreten ist.

```go
func TestGraphqlResolver(t *testing.T){
    table :=[]subTest{
        name string
        ...
    }{
        name: "retrieve_offer", 
        ...
    }

    for _, st := range table {
        t.Run(st.name, func(t *testing.T){
            ...
        })
    }
}

-> t.Run ein eigener Name
    -> st.name bekommt den Namen aus der table
    -> startet eine Testfunction (closure)

```

**complexere structure**

```go
type checker interface {
    check (*testing.T, string, string) bool    // vergleiche zwei Strings
}

type subTest struct {
    name  String
    shouldFail bool   // um auch den Fall abzudecken, dass Test einen fehler erzeugen muss, wenn nicht ist es ein Fehler
    checker    checker // parameter wie Werte verglichen werden (siehe interface konfiguration)
}

// now we can define different checker types
type checkGolden() struct { ...}

func (c checkGolden) check(t *testing.T, got, want string) bool {
    ...
}
```

**Mocking or faking**

```go
type DB interface {
    GetThing(string) (thing, error)
    ...
}

type mockDB struct {
    shouldFail bool
}

var errShouldFail = errors.New("db should fail")

func (m mochDB) GetThing(key string) (thing, error) {
    if m.shouldFail {
        return thing{}, fmt.Errorf("%s: %w", key, errShouldFail)
    }
    ...
}
```

**Main test function**

* setup test bevor die Test starten (init-Funktion)

```go
func TestMain(m *testing.M)  // testing.M
    stop, err := startEmulator()

    if err != nil {
        log.Println("*** Failed to start emulator ***")
        os.Exit(-1)
    }

    result := m.Run()  // run all unit tests

    stop()  // defer wird hier nicht verwendt, da man os.Exit(result) haben möchte
    os.Exit(result)
}
```


## philosophy

Gries & Conway

level of correctness 
1. it compiles [and passes static analysis]
2. if has no bugs that can be found just running the program
3. it works for some hand-picked test data
4. if works for typical, reasonable input
5. it works with test data chosen to be difficult
6. it works for all input that follows the specifications
7. if works for all valid inputs and likely error cases
8. it works for all input

works means it produced the desired behavior or fails safely

bis 4/5 meistens im Fokus


# Code Coverage

85-95% ok

```go
go test ./...                                          
go test ./... -cover                                   // coverage
go test ./... -coverprofile=c.out -covermode=count     // Ausgabe in Datei
    	go tool cover -html=c.out                      // Darstellung im Browser
``` 

## Test Automation: Gotests

https://github.com/cweill/gotests