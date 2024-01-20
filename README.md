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
// Вариант 1. Получаем словарь частот из исходной строки и из строки-шаблона. Проходим по ключам-значениям словаря частот dictTemplate, проверяем наличие символов и их количество (оно должно быть больше или равно) в dictStr
func checkStr(str: String, template: String) -> Bool {
    guard !str.isEmpty, str.count >= template.count else { return false }
     
    let dictStr = str.reduce(into: [:]) {$0[$1, default: 0] += 1}
    let dictTemplate = template.reduce(into: [:]) {$0[$1, default: 0] += 1}
    
    for (key, value) in dictTemplate {
        if dictStr[key, default: 0] < value {
            return false
        }
    }
    return true
}

// Вариант 2. Итерируем по проверяемой строке. Как только находим символ из шаблона, удаляем его из шаблона. Как только строка-шаблон станет пустой, возвращаем true.
func checkStr(str: String, template: String) -> Bool {
    guard !str.isEmpty, str.count >= template.count else { return false }
    
    var template = template
    for symbol in str {
        if template.contains(symbol) {
            template.remove(at: template.firstIndex(of: symbol)!)
            print(template)
        }
        if template.isEmpty {
            return true
        }
    }
    return false
}


var str = "AbTNIONdsKmOFtFYF"
var template = "TINKOFF" 

print(checkStr(str: str, template: template)) // true
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
---

Написать функцию, которая форматирует длительность, заданную в виде количества секунд, удобным для человека способом.

Функция должна принимать неотрицательное целое число. Если оно равно нулю, она просто возвращает "now". В противном случае продолжительность выражается в виде комбинации лет, дней, часов, минут и секунд.

Это гораздо проще понять на примере:

```
* For seconds = 62, your function should return 
    "1 minute and 2 seconds"
* For seconds = 3662, your function should return
    "1 hour, 1 minute and 2 seconds"
```

```swift
func formatDuration(_ seconds: Int) -> String {
    guard seconds > 0 else { return "now" }
    var (yearStr, dayStr, hourStr, minuteStr, secondStr) = ("", "", "", "", "")
    var remainder = 0
    var year = seconds / (365 * 24 * 60 * 60)
    remainder = seconds - year * 365 * 24 * 60 * 60
    var day = remainder / (24 * 60 * 60)
    remainder -= day * 24 * 60 * 60
    var hour = remainder / (60 * 60)
    remainder -= hour * 60 * 60
    var minute = remainder / 60
    let second = remainder - minute * 60

    switch second {
    case 0: secondStr = ""
    case 60:
        secondStr = ""
        minute += 1
    case 1 where (year + day + hour + minute) != 0: secondStr = " and \(second) second"
    case 1: secondStr = "\(second) second"
    case (1...59) where year + day + hour + minute != 0: secondStr = " and \(second) seconds"
    default: secondStr = "\(second) seconds"
    }

    switch minute {
    case 0: minuteStr = ""
    case 60:
        minuteStr = ""
        hour += 1
    case 1 where second == 0 && (year + day + hour) != 0: minuteStr = " and \(minute) minute"
    case 1 where (year + day + hour) != 0: minuteStr = ", \(minute) minute"
    case 1: minuteStr = "\(minute) minute"
    case (1...59) where second == 0 && (year + day + hour) != 0: minuteStr = " and \(minute) minutes"
    case (1...59) where (year + day + hour) != 0: minuteStr = ", \(minute) minutes"
    default: minuteStr = "\(minute) minutes"
    }

    switch hour {
    case 0: hourStr = ""
    case 24:
        hourStr = ""
        day += 1
    case 1 where (minute + second) == 0 && (year + day) != 0: hourStr = " and \(hour) hour"
    case 1 where (year + day) != 0: hourStr = ", \(hour) hour"
    case 1: hourStr = "\(hour) hour"
    case (1...23) where (minute + second) == 0 && (year + day) != 0: hourStr = " and \(hour) hours"
    case (1...23) where year + day != 0: hourStr = ", \(hour) hours"
    default: hourStr = "\(hour) hours"
    }

    switch day {
    case 0: dayStr = ""
    case 365:
        dayStr = ""
        year += 1
    case 1 where year != 0 && (hour + minute + second) == 0: dayStr = " and \(day) day"
    case 1 where year != 0: dayStr = ", \(day) day"
    case 1: dayStr = "\(day) day"
    case (1...364) where year != 0 && (hour + minute + second) == 0: dayStr = " and \(day) days"
    case (1...364) where year != 0: dayStr = ", \(day) days"
    default: dayStr = "\(day) days"
    }

    switch year {
    case 0: yearStr = ""
    case 1: yearStr = "\(year) year"
    default: yearStr = "\(year) years"
    }
    
    return yearStr + dayStr + hourStr + minuteStr + secondStr
}
```
---


Дан массив положительных или отрицательных целых чисел `I= [i1,..,in]`.
Вы должны получить сортированный массив P вида `[ [p, sum of all ij of I for which p is a prime factor (p positive) of ij] ...]`.
P будет отсортирован в порядке возрастания простых чисел (prime numbers).

Пример

```
I = [12, 15] # result = [(2, 12), (3, 27), (5, 15)]
```

[2, 3, 5] - это массив всех простых коэффициентов элементов I, отсюда и результат.

Может случиться так, что сумма равна 0, если некоторые числа отрицательны.
Пример: I = [15, 30, -45]. 
15, 30 и (-45) делятся на 5, поэтому 5 появляется в результате. 
Сумма чисел, для которых 5 является множителем, равна 0, поэтому в результате среди прочих чисел мы имеем [5, 0].

```swift
func sumOfDivided(_ l: [Int]) -> [(Int, Int)] {
    var result: [(Int, Int)] = []
    guard !l.isEmpty else { return result }

    func numberIsPrime(_ n: Int) -> Bool {
            guard n != 2 else { return true }
            guard n > 2 else { return false }

            var isPrime = true
            for i in 2...Int(sqrt(Double(n)).rounded()) {
                if (Double(n) / Double(i)).truncatingRemainder(dividingBy: 1) == 0 {
                    isPrime = false
                    break
                }
            }
            return isPrime
        }

    func findPrimeNumbersInTheRange(_ l: [Int]) -> [Int] {
        var result = [Int]()
        for number in 2...(l.map{abs($0)}.max()!) {
            if numberIsPrime(number) {
                result.append(number)
            }
        }
        return result
    }

    let primeNumbers = findPrimeNumbersInTheRange(l)
    var sum = 0
    var count = 0
    for primeNumber in primeNumbers {
        for number in l {
            if number % primeNumber == 0 {
                sum += number
                count += 1
            }
        }
        if count > 0 {
            result.append((primeNumber, sum))
            sum = 0
            count = 0
        }
    }
    return result
}
```
---


Дан двумерный массив и количество поколений, вычислите n временных шагов игры Конвея "Игра в жизнь".

Правила игры таковы:

Любая живая клетка с менее чем двумя живыми соседями умирает, как если бы это было вызвано перенаселением.
Любая живая клетка с более чем тремя живыми соседями погибает, как при перенаселении.
Любая живая клетка с двумя или тремя живыми соседями продолжает жить в следующем поколении.
Любая мертвая клетка с ровно тремя живыми соседями становится живой клеткой.
Соседями каждой клетки являются 8 клеток, расположенных непосредственно вокруг нее (т. е. окрестности Мура). 
Вселенная бесконечна в обоих измерениях x и y, и все клетки изначально мертвы - за исключением тех, которые указаны в аргументах. 
Возвращаемое значение должно представлять собой двумерный массив, обрезанный вокруг всех живых клеток. (Если живых клеток нет, то возвращается [[]]).

```swift
func getGeneration(_ cells: [[Int]], generations: Int) -> [[Int]] {
    
    guard !cells.isEmpty else { return [[]] }
    guard generations > 0 else { return cells }
    
    func createNewFild(_ field: [[Int]]) -> [[Int]] {
        var result: [[Int]] = []
        let rowSize = field[0].count
        for row in field {
            result.append([0, 0] + row + [0, 0])
        }
        let zeroArray = [Array(repeating: 0, count: rowSize + 4)]
        result = zeroArray + zeroArray + result + zeroArray + zeroArray
        return result
    }
    
    func transformField(_ field: [[Int]]) -> [[Int]] {
        var result = field
        for (i, row) in field.enumerated() where i > 0 && i < field.count - 1 {
            for (j, _) in row.enumerated() where j > 0 && j < row.count - 1 {
                let sumOfAlive = field[i - 1][j - 1] + field[i - 1][j] + field[i - 1][j + 1] + field[i][j + 1] + field[i + 1][j + 1] + field[i + 1][j] + field[i][j - 1] + field[i + 1][j - 1]
                switch sumOfAlive {
                case 0...1 where field[i][j] == 1: result[i][j] = 0
                case 2...3 where field[i][j] == 1: result[i][j] = 1
                case 3... where field[i][j] == 1: result[i][j] = 0
                case 3 where field[i][j] == 0: result[i][j] = 1
                default: break
                }
            }
        }
        return result
    }
    
    func cropArray(_ field: [[Int]]) -> [[Int]] {
        
        var minI = field.count
        var maxI = 0
        var minJ = field[0].count
        var maxJ = 0
        
        for (i, row) in field.enumerated() {
            for (j, _) in row.enumerated() {
                if field[i][j] == 1 {
                    minI = min(minI, i)
                    minJ = min(minJ, j)
                    maxI = max(maxI, i)
                    maxJ = max(maxJ, j)
                }
            }
        }
        
        var result: [[Int]] = Array(repeating: Array(repeating: 0, count: (maxJ - minJ + 1)), count: (maxI - minI + 1))
        
        for (i, row) in field.enumerated() where (minI...maxI).contains(i) {
            for (j, _) in row.enumerated() where (minJ...maxJ).contains(j) {
                if field[i][j] == 1 {
                    result[i - minI][j - minJ] = 1
                }
            }
        }
        return result
    }
    
    var field: [[Int]] = cells
    
    for _ in 1...generations {
        field = cropArray(transformField(createNewFild(field)))
    }
  
    return field
}
```
---


Кодирование десятичных чисел с помощью факториалов - это способ записи чисел в системе оснований, которая зависит от факториалов, а не от силы чисел.
В этой системе последняя цифра всегда равна 0 и находится в основании 0! Предыдущая цифра либо 0, либо 1 и имеет основание 1! Предыдущая цифра - 0, 1 или 2 и имеет основание 2! и т. д. В более общем случае предпоследняя цифра всегда равна 0, 1, 2, ..., n и находится в основании n!


Пример

Десятичное число 463 кодируется как "341010", потому что:

463 = 3×5! + 4×4! + 1×3! + 0×2! + 1×1! + 0×0!

Если мы ограничены цифрами 0...9, то самое большое число, которое мы можем закодировать, - 10!-1 (= 3628799). Поэтому мы расширим 0...9 буквами A...Z. 

Задача

Нам понадобятся две функции. Первая будет получать десятичное число и возвращать строку с факториальным представлением.

Вторая будет получать строку с факториальным представлением и выдавать десятичное представление.

Заданные числа всегда будут положительными.

```swift
func dec2FactString(_ nb: Int) -> String {
    var result = "0"
    var remainder = nb
    var index = 2
    while remainder != 0 {
        let number = remainder % index
        result += number < 10 ? String(number) : String(Character(UnicodeScalar(index + 54)!))
        remainder /= index
        index += 1
    }

    return String(result.reversed())
}

func factString2dDec(_ s: String) -> Int {

// Tail recursive factorial
    func factorial(_ n: Int) -> Int {
        func factorialN(_ n: Int, _ acc: Int) -> Int {
            return n == 0 ? acc : factorialN(n - 1, acc * n)
        }
        return factorialN(n, 1)
    }

    var result = 0

    for (index, letter) in s.reversed().enumerated() where letter != "0" {
        if ("A"..."Z").contains(letter) {
            result += Int(letter.asciiValue! - 55) * factorial(index)
        } else {
            result += Int(String(letter))! * factorial(index)
        }
    }
    return result
}
```
---


Напишите две функции, преобразующие римскую цифру в целое число и обратно.

Современные римские цифры записываются, выражая каждую цифру отдельно, начиная с самой левой цифры и пропуская любую цифру со значением ноль. 
Римскими цифрами 1990 обозначается так: 1000=M, 900=CM, 90=XC; в результате получается MCMXC. 
2008 год записывается как 2000=MM, 8=VIII; или MMVIII. В 1666 году каждый римский символ используется в порядке убывания: MDCLXVI.

Диапазон ввода: 1 <= n < 4000

```swift
class RomanNumerals {

  static func toRoman(_ number: Int) -> String {
      guard (1...3999).contains(number) else { return "Enter an integer number between 1 and 3999" }
      let arabicRomanicDictionary = [1: "I", 4: "IV", 5: "V", 9: "IX", 10: "X", 40: "XL", 50: "L", 90: "XC", 100: "C", 400: "CD", 500: "D", 900: "CM", 1000: "M"]
      var result = ""
      var tempNumber = number
      for (digit, letters) in arabicRomanicDictionary.sorted(by: {$1.key < $0.key}) {
          let count = tempNumber / digit
          result += String(repeating: letters, count: count)
          tempNumber %= digit
      }

      return result
  }
  
  static func fromRoman(_ roman: String) -> Int {
      let arabicRomanicDictionary = ["I": 1, "V": 5, "X": 10, "L": 50, "C": 100, "D": 500, "M": 1000]
      var result = 0
      var first = 0
      for letter in roman.reversed() {
          let second = arabicRomanicDictionary[String(letter)] ?? 0
          if second >= first {
              result += second
          } else {
              result -= second
          }
          first = second
      }
      return result
  }
}
```
---


Создайте функцию, которая принимает целое положительное число и возвращает следующее большее число, которое может быть образовано перестановкой его цифр. Например:

```
12 ==> 21
513 ==> 531
2017 ==> 2071
```

Если цифры нельзя переставить, чтобы получить большее число, верните nil.

```swift
func nextBigger(num: Int) -> Int? {
    guard num > 11 else { return nil }
    let arr = String(num).sorted()
    guard String(num).sorted() != String(num).reversed() else { return nil }
    var result = num
    var isEqual = false
    while !isEqual {
        result += 1
        if String(result).sorted() == arr {
            isEqual = true
        }
    }
  return result
}
```
---



Напишите функцию, которая принимает на вход целое число и возвращает количество битов, равных единице в двоичном представлении этого числа. 
Входные данные неотрицательны.

Пример: Двоичное представление числа 1234 - 10011010010, поэтому в данном случае функция должна вернуть 5.

```swift
func countBits(_ n: Int) -> Int {
  var result = 0
  for i in 0..<String(n, radix: 2).count {
    let mask = 1 << i
    if n & mask != 0 {
      result += 1
    }
  }
  return result
}
```
---


Даны два целых числа a, b, найдите их сумму, НО не разрешается использовать операторы + и -.

Числа (a,b) могут быть положительными, отрицательными значениями или нулями.

Возвращаемое значение будет целым числом.

```swift
func add(_ x: Int, _ y: Int) -> Int {
    guard x != 0 else { return y }
    guard y != 0 else { return x }
    var x = x
    var y = y
    while y != 0 {
        let shift = x & y
        x = x ^ y
        y = shift << 1
    }
    return x
}
```
---


Дана последовательность u, где u определяется следующим образом:

Число u(0) = 1.

Для каждого x в u, в последовательность добавляются y = 2 * x + 1 и z = 3 * x + 1.

Например: u = [1, 3, 4, 7, 9, 10, 13, 15, 19, 21, 22, 27, ...].
1 дает 3 и 4, 3 дает 7 и 10, 4 дает 9 и 13, 7 дает 15 и 22 и так далее...

Задача:

При заданном параметре n функция dblLinear() возвращает элемент u(n) упорядоченной по возрастанию последовательности u (в которой нет дубликатов).

Пример:

dbl_linear(10) должна вернуть 22

Алгоритм должен быть эффективным.

```swift
func dblLinear(_ n: Int) -> Int {
    guard n != 0 else { return 1 }
    var result = Array(repeating: 1, count: n + 1)
    var firstIndex = 0
    var secondIndex = 0
    
    for i in 1...n {
      result[i] = min(2 * result[firstIndex] + 1, 3 * result[secondIndex] + 1)
        if result[i] == (2 * result[firstIndex] + 1) {
            firstIndex += 1
        }
        if result[i] == (3 * result[secondIndex] + 1) {
            secondIndex += 1
        }
    }
    return result[n]
}
```






















