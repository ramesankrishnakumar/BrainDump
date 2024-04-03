### recursively look for proxies in just the file
grep "searchterm" file.txt

### recursively look for proxies in locations
grep -r -i -n --include="*.txt" ./ 'searchterm'
  grep : command
  -r  : recurssive
  -i : ignore-case
  --include : "*.txt" - all text files OR --include \*.txt
  ./ : start from current directory
  -n : relative line number in the file

### awk
awk '/types/ { print }' programming.txt
  search 'types' in `programming.txt` and prints the line
  default separator is "space" so awk '/types/ { print 0 }' programming.txt prints first column 

### sed - stream editor
sed 's/foo/bar/gI' test.txt > output1.txt
  's' : substitute 
  '/foo' : search 'foo'
  '/bar' : substitute 'bar'
