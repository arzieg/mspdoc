# regex

carfully use the regexp package for complex searches and validation (https://swtch.com/~rsc/regexp)

## simple search

```go
strings.HasPrefix(s, substr)
strings.HasSuffic(s, substr)
strings.Contains(s,substr)

strings.LastIndex(s, substr)
strings.LastIndexByte(s, char)

strings.Replace(s, substr, replacement, count)
strings.ReplaceAll(s, substr, replacement)
```

Bsp.:
```go
func B() string {
    _, file, line, _ := runtime.Caller(1)  // Umgebung des Prg.Aufrufs
    idx := strings.LastIndexByte(file, '/')
    return "=> "+file[idx+1:]+":"+strconv.Itoa(line)
}
```

## regex

```go
func main()
te := "aba abba abbba"
re := regexp.MustCompile("b+")
mm := re.FindAllString(te, -1)
id := re.FindAllStringIndex(te, -1)
fmt.Println(mm) // [b bb bbb]
fnt.Println(id) // [[1 2], [5 7] [10 13]]
for _, d := range id {
    fmt.Println(te[d[0]:d[1]]) // b bb bbb
}
```

Replacement bei Func
```go
func main(){
te := "aba abba abbba"
re := regexp.MustCompile("b+")
up := re.ReplaceAllStringsFunc(te, strings.ToUpper)
fmt.Println(up) // aBa aBBa aBBBa
}
```

see: https://golang.org/pkg/regexp/syntax

```go
[a-z]  // suche nach a-z in ascii!
[[:alpha:]]  // ist alphabet (unicode aware)
[[:alnum:]]  // alphanumeric
[[:punct:]]  // punctuation 
[[:print:]]  // printable character
[[:xdigit]]  // any hexadecimal char
```

