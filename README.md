# SolvingTasksInSwift

В программе используются большие числа целые положительные числа, больше, чем UInt.max, поэтому они хранятся в виде массивов цифр. Написать функцию, которая суммирует два таких числа, принимает их в виде массивов цифр и возвращает массив цифр.

```swift
func sumLargeNumbers (firstArr: [Int], secondArr: [Int]) -> [Int] {
    guard !firstArr.isEmpty, !secondArr.isEmpty else { return [] }
    var result: [Int] = []
    var decimalPlace = 0 // Количество десятков. Здесь либо 0 либо 1
    for i in stride(from: firstArr.count - 1, through: 0, by: -1) {
        var sum = firstArr[i] + secondArr[i] + decimalPlace
        decimalPlace = sum / 10 // Если сумма 10 и выше, то будет 1, иначе 0
        result.append(sum % 10)
    }
    if decimalPlace != 0 { // После цикла проверяем, не нужно ли нам добавить ещё один разряд в результирующее число
        result.append(decimalPlace)
    }
    return result.reversed()
}

var arr1: [Int] = [1, 2]
var arr2: [Int] = [8, 8]

print(sumLargeNumbers(firstArr: arr1, secondArr: arr2)) // [1, 0, 0]
```
---


Дан массив целых чисел. Некоторые числа встречаются в массиве по нескольку раз. Написать функцию, которая вынимает из массива k наиболее часто встречающихся элементов. 

```swift
// Вариант 1. Создаём словарь, где ключами будут элементы массива, а значениями - их частоты. Далее делаем сортировку по значениям и из сортированного массива берём первые k элементов.

func findTheMostCommonElements (array: [Int], k: Int) -> [Int] {
    guard !array.isEmpty, k > 0 else { return [] }
    var result: [Int] = []
    
// Получаем словарь с частотами
    let frequenciesOfElements = array.reduce(into: [:]) { $0[$1, default: 0] += 1 }
    print(frequenciesOfElements) // [2: 4, 4: 1, 1: 2, 5: 1, 3: 2]
    
// Сортируем пары "ключ-значение"
    let sortedArr = frequenciesOfElements.sorted(by: { $0.value > $1.value })
    print(sortedArr) // [(key: 2, value: 4), (key: 1, value: 2), (key: 3, value: 2), (key: 4, value: 1), (key: 5, value: 1)]
    
// Выбираем из массива sortedArr k первых элементов. Это и будут значения с самыми большими частотами. k могут задать больше максимального индекса sortedArr (т.е. количества уникальных элементов в исходном массиве). Следим, чтобы индекс i не привысил его.
    for i in 0..<min(k, sortedArr.count) {
        result.append(sortedArr[i].key)
    }
    return result
}

// Второй вариант. Вместо создания словаря с частотами создаём массив, где индексом служит исходное число, а значение - его частота. Остальные значения массива - нули.
func findTheMostCommonElements (array: [Int], k: Int) -> [Int] {
    guard !array.isEmpty, k > 0 else { return [] }
    var result: [Int] = []
    
    // Создаём массив, размером с максимальный элемент исходного массива и заполняем его нулями
    var tempArr = Array(repeating: 0, count: (array.max() ?? 0) + 1)
    
    // Заполняем значения нового массива частотами по индексам - элементам исходного массива
    for element in array {
        tempArr[element] += 1
    }
    print(tempArr) // [0, 2, 4, 2, 1, 1] - массив частот

    // k раз находим значение с самой максимальной частотой в массиве tempArr. Найденные значения при повторном поиске игнорируем.
    for _ in 0..<k { 
        var maxFrequency = 1 // Нас интересуют только элементы, которые встречаются хотя бы один раз, ноль не подходит
        var mostFrequentValue = 0

        for (i, frequency) in tempArr.enumerated() where !result.contains(i) {
            if frequency > maxFrequency {
                maxFrequency = frequency
                mostFrequentValue = i
            }
        }
        if mostFrequentValue > 0 {
            result.append(mostFrequentValue)
        }
    }
    return result
}

let items = [1, 2, 3, 4, 5, 3, 2, 1, 2, 2]
print(findTheMostCommonElements(array: items, k: 3)) // [2, 1, 3]
```
---


Дана строка. Проверить, есть ли в ней буквы из которых можно собрать слово "TINKOFF"

```swift
func checkStr(str: String, template: String) -> Bool {
    guard !str.isEmpty, str.count < template.count else { return false }
    
    // Получаем словарь частот из исходной строки и из строки-шаблона
    let dictStr = str.reduce(into: [:]) {$0[$1, default: 0] += 1}
    let dictTemplate = template.reduce(into: [:]) {$0[$1, default: 0] += 1}
    
    // Проходим по ключам-значениям словаря частот dictTemplate, проверяем наличие символов и их количество (оно должно быть больше или равно) в dictStr
    for (key, value) in dictTemplate {
        if dictStr[key, default: 0] < value {
            return false
        }
    }
    return true
}

var str = "AbTNIONdsKmOFtFYF"
var template = "TINKOFF" 

print(checkStr(str: str, template: template)) // true
```
---


Составьте цепочку из домино.
Напишите функцию, которая упорядочивает заданный набор домино таким образом, чтобы они образовали правильную цепочку (точки на одной половине камня совпадают с точками на соседней половине соседнего камня) и чтобы точки на половинках камней, у которых нет соседей (первый и последний камень), совпадали друг с другом.

Например, для домино [2|1], [2|3] и [1|3] вы должны получить что-то вроде [1|2] [2|3] [3|1] или [3|2] [2|1] [1|3] или [1|3] [3|2] [2|1] и т. д., где первое и последнее числа одинаковы.
Для домино [1|2], [4|1] и [2|3] полученная цепочка не верна: первые и последние числа [4|1] [1|2] [2|3] не совпадают. 4 != 3

```swift
func isChained(_ input: [(Int, Int)]) -> Bool {
    guard !input.isEmpty else { return false }
    if input.count == 1 {
        if input[0].0 != input[0].1 {
            return false
        } else {
            return true
        }
    }
    if input.filter({ $0.0 == $0.1 }).count == input.count {
        return false
    }
    var uniqNumbers = Set<Int>()
    uniqNumbers = input.reduce(into: uniqNumbers){ $0.insert($1.0) }
    uniqNumbers = input.reduce(into: uniqNumbers){ $0.insert($1.1) }
    for number in uniqNumbers {
        let total = input.filter{ $0.0 == number }.count + input.filter{ $0.1 == number }.count
        if total % 2 == 1 || input.filter({ $0.0 == $0.1 }).count == total / 2 {
            return false
        } else {
            for domino in input.filter({ $0.0 == number }) {
                if input.filter({ $0.0 == domino.1 && $0.1 == domino.0 }).count == total / 2 {
                    return false
                }
            }
        }
    }
    return true
}

struct Dominoes {

    let dominoes: [(Int, Int)]

    var chained: Bool {
        guard !dominoes.isEmpty else { return false }
        if dominoes.count == 1 {
            if dominoes[0].0 != dominoes[0].1 {
                return false
            } else {
                return true
            }
        }
        if dominoes.filter({ $0.0 == $0.1 }).count == dominoes.count {
            return false
        }
        var uniqNumbers = Set<Int>()
        uniqNumbers = dominoes.reduce(into: uniqNumbers){ $0.insert($1.0) }
        uniqNumbers = dominoes.reduce(into: uniqNumbers){ $0.insert($1.1) }
        for number in uniqNumbers {
            let total = dominoes.filter{ $0.0 == number }.count + dominoes.filter{ $0.1 == number }.count
            if total % 2 == 1 || dominoes.filter({ $0.0 == $0.1 }).count == total / 2 {
                return false
            } else {
                for domino in dominoes.filter({ $0.0 == number }) {
                    if dominoes.filter({ $0.0 == domino.1 && $0.1 == domino.0 }).count == total / 2 {
                        return false
                    }
                }
            }
        }
        return true
    }

    init(_ dominoes: [(Int, Int)]) {
    self.dominoes = dominoes
    }

}
```
---


Возьмите следующий IPv4-адрес: 128.32.10.1
Этот адрес состоит из 4 октетов, где каждый октет - это один байт (или 8 бит).
Первый октет 128 имеет двоичное представление: 10000000
2-й октет 32 имеет двоичное представление: 00100000
3-й октет 10 имеет двоичное представление: 00001010
4-й октет 1 имеет двоичное представление: 00000001
Таким образом, 128.32.10.1 == 10000000.00100000.00001010.00000001

Поскольку приведенный выше IP-адрес состоит из 32 бит, мы можем представить его как беззнаковое 32-битное число: 2149583361.

Напешите функцию, которая принимает беззнаковое 32-битное число и возвращает строковое представление его IPv4-адреса.

```swift
func ipv4(of i32: UInt32) -> String {
    var result = [String]()
    var str = String(i32, radix: 2)
    str = String(repeating: "0", count: 32 - str.count) + str
    var count = 0
    var number = ""
    for digit in str {
        count += 1
        if count == 8 {
            number += String(digit)
            result.append(String(Int(number, radix: 2)!))
            number = ""
            count = 0
        } else {
            number += String(digit)
        }
    }
    return result.joined(separator: ".")
}
```
---


Создать спираль NxN заданного размера.
Например, для N=10

```
0000000000
.........0
00000000.0
0......0.0
0.0000.0.0
0.0..0.0.0
0.0....0.0
0.000000.0
0........0
0000000000
```

Возвращаемое значение должно содержать массив массивов, состоящих из 0 и 1, первая строка которого состоит из 1. Например, для заданного размера 5 результат должен быть таким:
`[[1,1,1,1,1],[0,0,0,0,1],[1,1,1,0,1],[1,0,0,0,1],[1,1,1,1,1]]` 

Из-за крайних случаев для крошечных спиралей размер будет не меньше 5.
Общее правило гласит, что змейка, сделанная с помощью '1', не может касаться сама себя.

```swift
func spiral(_ n : Int) -> [[Int]] {
    var result = Array(repeating: Array(repeating: 0, count: n), count: n)

    for i in 0..<n {
        result[0][i] = 1
    }

    for index in stride(from: 0, through: (n - 1) / 2, by: 2) {
        for j in index...(n - 1 - index) {
            result[j][n - 1 - index] = 1
            if result[n - 2 - index][j] != 1 {
            result[n - 1 - index][j] = 1
            }
        }
        if (index + 2)<=(n / 2) {
            for j in (index + 2)...(n - 1 - index) {
                result[j][index] = 1
                if result[index + 3][j - 2] != 1 {
                    result[index + 2][j - 2] = 1
                }
            }
        }
    }
    return result
}
```
---


Напишите функцию, которая принимает две квадратные (NxN) матрицы (двумерные массивы) и возвращает их произведение. Задаются только квадратные матрицы.
Как перемножить две квадратные матрицы A и B: чтобы заполнить ячейку [n][m] матрицы C (результат), нужно сначала умножить элементы n-й строки матрицы A 
на элементы m-го столбца матрицы B, а затем взять сумму всех этих произведений. 

Пример:

```
  A         B          C
|1 2|  x  |3 2|  =  | 5 4|
|3 2|     |1 1|     |11 8|
```

```swift
func matrixMultiplication(_ a:[[Int]], _ b:[[Int]]) -> [[Int]] {
    guard a.count == b.count, !a.isEmpty, !b.isEmpty else { return [[Int]]() }
    var result = Array(repeating: Array(repeating: 0, count: a.count), count: a.count)

    for (i, row) in result.enumerated() {
        for (j, _) in row.enumerated() {
            var value = 0
            for index in 0..<a.count {
                value += a[i][index] * b[index][j]
            }
            result[i][j] = value
        }
    }
    return result
}
```
---


Существует секретная строка, которая вам неизвестна. Получив набор случайных триплетов из этой строки, восстановите исходную строку.
Под триплетом здесь понимается последовательность из трех букв, причем каждая буква встречается где-то перед следующей в данной строке. 
Например "whi" - это триплет для строки "whatisup".

В качестве упрощения можно предположить, что ни одна буква не встречается в секретной строке более одного раза.
Вы можете ничего не предполагать о предоставленных вам триплетах, кроме того, что это правильные триплеты и что они содержат достаточно информации для вывода исходной строки. 
В частности, это означает, что секретная строка никогда не будет содержать букв, которые не встречаются ни в одном из предоставленных вам триплетов.

```swift
func recoverSecret(from triplets: [[String]]) -> String {
    guard !triplets.isEmpty else { return "" }
    guard triplets.count != 1 else { return (triplets.first!).joined() }
    var letters = Array(Set(triplets.flatMap{ $0 }))
    var result = ""
    while letters.count > 0 {
        var min = letters[0]
        var flag = true
        while flag {
            loop: for row in triplets where row.contains(min) {
                switch row.firstIndex(of: min)! {
                case 1:
                    if !result.contains(row[0]) {
                        min = row[0]
                        flag = true
                        break loop
                    } else {
                        flag = false
                    }
                case 2:
                    if !result.contains(row[0]) {
                        min = row[0]
                        flag = true
                        break loop
                    } else if !result.contains(row[1]) {
                        min = row[1]
                        flag = true
                        break loop
                    } else {
                        flag = false
                    }
                default: flag = false
                }
            }
        }
        result.append(min)
        letters.remove(at: letters.firstIndex(of: min)!)
    }
    return result
}
```
---


Напишите функцию AlternatingSplit(), которая берет один связанный список (Linked List) и делит его узлы на два меньших списка. 
Вложенные списки должны быть составлены из чередующихся элементов исходного списка. 
Так, если исходный список имеет вид a -> b -> a -> b -> a -> null, то один подсписок должен быть a -> a -> a -> null, а другой - b -> b -> null.

Для простоты мы используем объект Context для хранения и возврата состояния двух связанных списков.
Объект Context, содержащий два мутировавших списка, должен быть возвращен функцией AlternatingSplit().

Если переданный головной узел является nil или одиночным узлом, произойдет ошибка.

```
var list = 1 -> 2 -> 3 -> 4 -> 5 -> null
alternatingSplit(list).first === 1 -> 3 -> 5 -> null
alternatingSplit(list).second === 2 -> 4 -> null
```

```swift
class Node {
    var data: Int
    var next: Node?
    init(_ data: Int) {
        self.data = data
    }
}

struct Context {
    var source: Node?
    var dest: Node?
    mutating func appendSource(_ node: Node?) {
        if var head = self.source {
            while head.next != nil {
                head = head.next!
            }
            head.next = node
        } else {
            self.source = node
        }
    }
    mutating func appendDest(_ node: Node?) {
        if var head = self.dest {
            while head.next != nil {
                head = head.next!
            }
            head.next = node
        } else {
            self.dest = node
        }
    }
}

func alternatingSplit(head : Node?) throws -> Context? {
    guard head != nil else { throw AlternatingSplitError.listIsEmpty }
    guard head?.next != nil else { throw AlternatingSplitError.listHasOnlyOneNode }
    var result = Context(source: Node(head!.data), dest: Node(head!.next!.data))
    var currentNode: Node? = head?.next?.next
    var flag = true
    while currentNode != nil {
        if flag {
            result.appendSource(Node(currentNode!.data))
            flag.toggle()
        } else {
            result.appendDest(Node(currentNode!.data))
            flag.toggle()
        }
        currentNode = currentNode?.next
    }
    return result
}

enum AlternatingSplitError: Error {
    case listIsEmpty
    case listHasOnlyOneNode
}
```
---


Семен Николаевич Корсаков (изобретатель перфокарт) прислал вам сообщение из 1832 года. Он придумал, как отправить сообщение в будущее, но еще не освоил новые способы передачи. 
Мы получили от него две перфокарты. На первой карте, насколько я могу судить, есть расшифровка, а на второй - сообщение.

Давайте попробуем создать два метода шифрования и дешифрования, чтобы не оставить нашего нового друга без ответа.

Вы можете использовать предварительно загруженный словарь, называемый шифром. 

Зона нажатия заменена на массивы "A" и "B".
Чтобы прочитать символ, нужно заменить соответствующие буквы и/или цифры на пустую строку " " (она же дыра).
Если весь символьный ряд находится на своем месте, то это пробел в строке.
Длина сообщения может достигать 80 символов.

Пример

```
[[" ", " ", "B", "B", "B"],
 ["A", "A", " ", " ", " "],
 ["0", "0", "0", "0", "0"],
 ["1", "1", "1", "1", "1"],
 ["2", "2", "2", "2", "2"],
 ["3", "3", " ", " ", "3"],
 ["4", "4", "4", "4", "4"],
 ["5", " ", "5", "5", "5"],
 ["6", "6", "6", "6", " "],
 ["7", "7", "7", "7", "7"],
 [" ", "8", "8", "8", "8"],
 ["9", "9", "9", "9", "9"]] // should return "HELLO"
```

```swift
func fromPerfoCard(_ card: [[Character]]) -> String {
    let template: [Character] = ["B", "A", "0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
    var result = ""
    for index in 0...79 {
        var value = [Character]()
        for (i, row) in card.enumerated() {
            if index < row.count {
                if row[index] != template[i] {
                    value.append(template[i])
                }
            }
        }
        result.append(encryption.first(where: { $0.value == value })?.key ?? " ")
    }
    return result.trimmingCharacters(in: .whitespaces)
}

func toPerfoCard(_ message: String) -> [[Character]] {
    let message = message.map{ $0 }
    let template: [Character] = ["B", "A", "0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
    var result = [[Character]]()
    for char in template {
        result.append(Array(repeating: char, count: message.count))
    }
    for (index, symbol) in message.enumerated() {
        if let hole = encryption[symbol] {
            for position in hole {
                for (i, row) in result.enumerated() {
                    if row[index] == position {
                        result[i][index] = " "
                    }
                }
            }
        }
    }
    return result
}
```
---


Напишите функцию, которая получает две строки и возвращает n, где n равно количеству символов, на которое нужно сдвинуть первую строку вперед, чтобы она совпала со второй. 
Проверка должна быть чувствительна к регистру.

Например, возьмем строки "fatigue" и "tiguefa". В этом случае первая строка была сдвинута на 5 символов вперед, чтобы получить вторую строку, поэтому будет возвращено 5.

Если вторая строка не является корректным вращением первой строки, метод возвращает nil.

```swift
func shiftedDiff(_ s1: String, _ s2: String) -> Int? {
    guard s1.count == s2.count && !s1.isEmpty && !s2.isEmpty else { return nil }
    guard s1 != s2 else { return 0 }
    var result: Int?
    var count = 0
    var tempS1 = s1
    while count < s1.count - 1 {
        count += 1
        let lastLetter = tempS1.removeLast()
        tempS1 = String(lastLetter) + tempS1
        if tempS1 == s2 {
            result = count
            break
        }
    }
    return result
}
```
---


Необходимо написать алгоритм, который уплощает структуру массивов до двух уровней. 
Это означает, что каждый массив, находящийся на втором уровне, будет слит со своим родителем. Сохраняются только два уровня.

Пустые массивы игнорируются.

```
var array = [1, [2, 3], [4, 5, [6, 7, 8], 9, 10, [11, [12, [13, 14], 15], 16], 17], 18];
flattenTwoLevels(array); // should return [1,[2,3],[4,5,6,7,8,9,10,11,12,13,14,15,16,17], 18]

flattenTwoLevels([1,[2, 3, [], [4, [], 5]]]) // should return [1,[2,3,4,5]]
```

```swift
func flattenedArray(array: [Any]) -> [Int] {
    var result = [Int]()
    for element in array {
        if element is Int {
            result.append(element as! Int)
        } else if element is [Any] {
            let value = flattenedArray(array: element as! [Any])
                for number in value {
                    result.append(number)
            }
        }
    }
    return result
}

func flattenTwoLevels(_ arr: [Any]) -> [Any] {
    var result = [Any]()
    for element in arr {
        if element is Int {
            result.append(element)
        } else if element is [Int] {
            result.append(element)
        } else if element is [Any] {
            let value = flattenedArray(array: element as! [Any])
            var newResult = [Int]()
                for i in value {
                    newResult.append(i)
            }
            result.append(newResult)
        }
    }
    return result
}
```
---


Создайте функцию parse, принимающую ровно один аргумент типа String, который представляет собой строковое представление связанного списка. 
Ваша функция должна возвращать соответствующий связанный список, построенный из экземпляров класса Node. 

Строковое представление списка имеет следующий формат: значение узла, затем пробел, стрелка и еще один пробел (" -> "), затем остальная часть связанного списка. 
Каждое строковое представление связанного списка будет заканчиваться символом "nil" 

```swift
func parse(_ str: String) -> Node? {
    guard str != "nil" else { return nil }
    let array = str.components(separatedBy: " -> ").compactMap{Int($0)}
    var node = Node(array.last!, nil)
    guard array.count != 1 else { return node }
    for i in stride(from: (array.count - 2), through: 0, by: -1) {
        node = Node(array[i], node)
    }
    return node
}
```






