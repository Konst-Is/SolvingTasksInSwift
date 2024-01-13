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
Вычислите способ упорядочить заданный набор домино таким образом, чтобы они образовали правильную цепочку (точки на одной половине камня совпадают с точками на соседней половине соседнего камня) и чтобы точки на половинках камней, у которых нет соседей (первый и последний камень), совпадали друг с другом.
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

`
  A         B          C
|1 2|  x  |3 2|  =  | 5 4|
|3 2|     |1 1|     |11 8|
`

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






