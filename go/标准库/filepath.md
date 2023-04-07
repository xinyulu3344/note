# filepath

```go
// 判断path是否是绝对路径
func IsAbs(path string) bool

// Abs函数返回path代表的绝对路径。
// 如果path不是绝对路径，会加入当前工作目录以使之成为绝对路径。
// 因为硬链接的存在，不能保证返回的绝对路径是唯一指向该地址的绝对路径。
func Abs(path string) (string, error)

// Rel函数返回一个相对路径。
// 将basepath和该路径用路径分隔符连起来的新路径，在词法上等价于targpath。
// 也就是说，Join(basepath, Rel(basepath, targpath))等价于targpath本身。
// 如果成功执行，返回值总是相对于basepath的，即使basepath和targpath没有共享的路径元素。
// 如果两个参数一个是相对路径而另一个是绝对路径，或者targpath无法表示为相对于basepath的路径，将返回错误。
func Rel(basepath, targpath string) (string, error)

// 将PATH或GOPATH等环境变量里的多个路径分割开（这些路径被OS特定的表分隔符连接起来）。
// 与strings.Split函数的不同之处是：对""，SplitList返回[]string{}，而strings.Split返回[]string{""}。
func SplitList(path string) []string

// Split函数将路径从最后一个路径分隔符后面位置分隔为两个部分（dir和file）并返回。
// 如果路径中没有路径分隔符，函数返回值dir会设为空字符串，file会设为path。两个返回值满足path == dir+file。
func Split(path string) (dir, file string)

// Join函数可以将任意数量的路径元素放入一个单一路径里，会根据需要添加路径分隔符。
// 结果是经过简化的，所有的空字符串元素会被忽略。
func Join(elem ...string) string

// FromSlash函数将path中的斜杠（'/'）替换为路径分隔符并返回替换结果，多个斜杠会替换为多个路径分隔符。
func FromSlash(path string) string

// ToSlash函数将path中的路径分隔符替换为斜杠（'/'）并返回替换结果，多个路径分隔符会替换为多个斜杠。
func ToSlash(path string) string

// VolumeName函数返回最前面的卷名。
// 如Windows系统里提供参数"C:\foo\bar"会返回"C:"；
// 提供"\\host\share\foo" 返回 "\\host\share"
// 其他平台会返回""。
func VolumeName(path string) (v string)

// Dir返回路径除去最后一个路径元素的部分，即该路径最后一个元素所在的目录。
// 在使用Split去掉最后一个元素后，会简化路径并去掉末尾的斜杠。
// 如果路径是空字符串，会返回"."；
// 如果路径由1到多个路径分隔符后跟0到多个非路径分隔符字符组成，会返回单个路径分隔符；
// 其他任何情况下都不会返回以路径分隔符结尾的路径。
func Dir(path string) string

// Base函数返回路径的最后一个元素。
// 在提取元素前会去掉末尾的路径分隔符。
// 如果路径是""，会返回"."；
// 如果路径是只有一个斜杆构成，会返回单个路径分隔符。
func Base(path string) string

// Ext函数返回path文件扩展名。
// 返回值是路径最后一个路径元素的最后一个'.'起始的后缀（包括'.'）。
// 如果该元素没有'.'会返回空字符串。
func Ext(path string) string

// Clean函数通过单纯的词法操作返回和path代表同一地址的最短路径。
// 它会不断的依次应用如下的规则，直到不能再进行任何处理：
// 1. 将连续的多个路径分隔符替换为单个路径分隔符
// 2. 剔除每一个.路径名元素（代表当前目录）
// 3. 剔除每一个路径内的..路径名元素（代表父目录）和它前面的非..路径名元素
// 4. 剔除开始一个根路径的..路径名元素，即将路径开始处的"/.."替换为"/"（假设路径分隔符是'/'）
func Clean(path string) string

// EvalSymlinks函数返回path指向的符号链接（软链接）所包含的路径。
// 如果path和返回值都是相对路径，会相对于当前目录；
// 只要path和返回值中，有一个是绝对路径，就会返回绝对路径
func EvalSymlinks(path string) (string, error)

// Match要求匹配整个name字符串，而不是它的一部分。只有pattern语法错误时，会返回ErrBadPattern。
// Windows系统中，不能进行转义：'\\'被视为路径分隔符。
//	pattern:
//		{ term }
//	term:
//		'*'         匹配0或多个非路径分隔符的字符
//		'?'         匹配1个非路径分隔符的字符
//		'[' [ '^' ] { character-range } ']'
//		            字符组（必须非空）
//		c           匹配字符 c (c != '*', '?', '\\', '[')
//		'\\' c     匹配字符 c
//
//	character-range:
//		c            匹配字符c (c != '\\', '-', ']')
//		'\\' c      匹配字符c
//		lo '-' hi   匹配区间[lo, hi]内的字符
func Match(pattern, name string) (matched bool, err error)

// Glob函数返回所有匹配模式匹配字符串pattern的文件或者nil（如果没有匹配的文件）。
// pattern的语法和Match函数相同。pattern可以描述多层的名字，如/usr/*/bin/ed（假设路径分隔符是'/'）。
func Glob(pattern string) (matches []string, err error)

// Walk函数对每一个文件/目录都会调用WalkFunc函数类型值。
// 调用时path参数会包含Walk的root参数作为前缀；
// 就是说，如果Walk函数的root为"dir"，该目录下有文件"a"，将会使用"dir/a"调用walkFn参数。
// walkFn参数被调用时的info参数是path指定的地址（文件/目录）的文件信息，类型为os.FileInfo。
// 如果遍历path指定的文件或目录时出现了问题，传入的参数err会描述该问题，
// WalkFunc类型函数可以决定如何去处理该错误（Walk函数将不会深入该目录）；
// 如果该函数返回一个错误，Walk函数的执行会中止；
// 只有一个例外，如果Walk的walkFn返回值是SkipDir，将会跳过该目录的内容而Walk函数照常执行处理下一个文件。
type WalkFunc

// Walk函数会遍历root指定的目录下的文件树，对每一个该文件树中的目录和文件都会调用walkFn，包括root自身。
// 所有访问文件/目录时遇到的错误都会传递给walkFn过滤。
// 文件是按词法顺序遍历的，这让输出更漂亮，但也导致处理非常大的目录时效率会降低。
// Walk函数不会遍历文件树中的符号链接（快捷方式）文件包含的路径。
func Walk(root string, walkFn WalkFunc) error

// HasPrefix函数出于历史兼容问题保留，不应被使用。
func HasPrefix(p, prefix string) bool
```

