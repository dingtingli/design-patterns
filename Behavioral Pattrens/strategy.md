# 策略模式 strategy

## 极客时间笔记

>Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

>定义一系列的算法，对每一个进行封装，使得它们可以相互替换。策略模式让算法可以独立于使用它的客户端进行变化。

策略模式解耦的是策略的定义、创建、使用这三部分。

### 定义

```c#
public interface IStrategy 
{ 
    void AlgorithmInterface(); 
}

public class ConcreteStrategyA : IStrategy 
{ 
    public void AlgorithmInterface() 
    { 
        //具体的算法... 
    } 
}

public class ConcreteStrategyB : IStrategy 
{ 
    public void AlgorithmInterface() 
    { 
        //具体的算法... 
    } 
}
```

### 创建

```c#
using System.Collections.Generic;

public class StrategyFactory 
{
    private static readonly Dictionary<string, IStrategy> strategies = new Dictionary<string, IStrategy>()
    {
        { "A", new ConcreteStrategyA() },
        { "B", new ConcreteStrategyB() }
    };

    public static IStrategy GetStrategy(string type)
    {
        if (string.IsNullOrEmpty(type))
        {
            throw new ArgumentException("type should not be empty.");
        }
        return strategies[type];
    }
}

```

### 使用

客户端代码一般如何确定使用哪个策略呢？最常见的是运行时动态确定使用哪种策略，这也是策略模式最典型的应用场景。


这里的“运行时动态”指的是，我们事先并不知道会使用哪个策略，而是在程序运行期间，根据配置、用户输入、计算结果等这些不确定因素，动态决定使用哪种策略


讲到使用状态模式来避免分支判断逻辑的时候，你会发现，它们使用的是同样的套路。本质上都是借助“查表法”，根据 type 查表（代码中的 strategies 就是表）替代根据 type 分支判断。

### 示例

会通过一步一步地分析、重构，给你展示一个设计模式是如何“创造”出来的。设计原则和思想其实比设计模式更加普适和重要，掌握了代码的设计原则和思想，我们甚至可以自己创造出来新的设计模式。

需求：

1. 假设有这样一个需求，希望写一个小程序，实现对一个文件进行排序的功能。文件中只包含整型数，并且，相邻的数字通过逗号来区隔。如果由你来编写这样一个小程序，你会如何来实现呢？

2. 如果文件很大呢？比如有 10GB 大小，因为内存有限（比如只有 8GB 大小），我们没办法一次性加载文件中的所有数据到内存中，这个时候，我们就要利用外部排序算法。

3. 如果文件更大，比如有 100GB 大小，我们为了利用 CPU 多核的优势，可以在外部排序的基础之上进行优化，加入多线程并发排序的功能，这就有点类似“单机版”的 MapReduce。

4. 如果文件非常大，比如有 1TB 大小，即便是单机多线程排序，这也算很慢了。这个时候，我们可以使用真正的 MapReduce 框架，利用多机的处理能力，提高排序的效率。

如何将解决思路翻译成代码实现？


bing

```c#
using System;
using System.Collections.Generic;
using System.IO;

namespace FileSorter
{
    class Program
    {
        static void Main(string[] args)
        {
            string inputFilePath = "input.txt";
            string outputFilePath = "output.txt";

            // Step 1: Read the input file and split it into chunks
            List<string> chunkFilePaths = new List<string>();
            int chunkSize = 1000000; // 1 MB
            using (StreamReader reader = new StreamReader(inputFilePath))
            {
                int i = 0;
                while (!reader.EndOfStream)
                {
                    List<int> chunk = new List<int>();
                    while (chunk.Count < chunkSize && !reader.EndOfStream)
                    {
                        string line = reader.ReadLine();
                        if (line != null)
                        {
                            string[] values = line.Split(',');
                            foreach (string value in values)
                            {
                                if (int.TryParse(value, out int number))
                                {
                                    chunk.Add(number);
                                }
                            }
                        }
                    }

                    if (chunk.Count > 0)
                    {
                        chunk.Sort();
                        string chunkFilePath = $"chunk_{i}.txt";
                        using (StreamWriter writer = new StreamWriter(chunkFilePath))
                        {
                            foreach (int number in chunk)
                            {
                                writer.WriteLine(number);
                            }
                        }
                        chunkFilePaths.Add(chunkFilePath);
                        i++;
                    }
                }
            }

            // Step 2: Merge the sorted chunks into a single output file
            using (StreamWriter writer = new StreamWriter(outputFilePath))
            {
                List<StreamReader> readers = new List<StreamReader>();
                foreach (string chunkFilePath in chunkFilePaths)
                {
                    readers.Add(new StreamReader(chunkFilePath));
                }

                SortedList<int, StreamReader> heap = new SortedList<int, StreamReader>();
                foreach (StreamReader reader in readers)
                {
                    if (!reader.EndOfStream)
                    {
                        int number = int.Parse(reader.ReadLine());
                        heap.Add(number, reader);
                    }
                }

                while (heap.Count > 0)
                {
                    KeyValuePair<int, StreamReader> pair = heap.First();
                    heap.RemoveAt(0);

                    int number = pair.Key;
                    StreamReader reader = pair.Value;
                    writer.WriteLine(number);

                    if (!reader.EndOfStream)
                    {
                        number = int.Parse(reader.ReadLine());
                        heap.Add(number, reader);
                    }
                }

                foreach (StreamReader reader in readers)
                {
                    reader.Close();
                }
            }

            // Step 3: Delete the temporary chunk files
            foreach (string chunkFilePath in chunkFilePaths)
            {
                File.Delete(chunkFilePath);
            }

            Console.WriteLine("Done!");
        }
    }
}
```

This implementation reads the input file and splits it into chunks of a fixed size. Each chunk is sorted and written to a temporary file. The sorted chunks are then merged into a single output file using a heap data structure.

To handle larger files, you could increase the size of the chunks or use an external sorting algorithm like merge sort. To handle even larger files, you could use multi-threading or distributed computing techniques like MapReduce.

chatgpt 4

1. 假设有这样一个需求，希望写一个小程序，实现对一个文件进行排序的功能。文件中只包含整型数，并且，相邻的数字通过逗号来区隔。如果由你来编写这样一个小程序，你会如何来实现呢？

有限文件排序：可以简单地通过读取整个文件，然后使用内部排序算法来排序整个文件。这可以通过Python实现如下：

```python
def sort_small_file(filename):
    with open(filename, 'r') as file:
        numbers = list(map(int, file.read().split(',')))
    numbers.sort()
    with open(filename, 'w') as file:
        file.write(','.join(map(str, numbers)))

```


2. 如果文件很大呢？比如有 10GB 大小，因为内存有限（比如只有 8GB 大小），我们没办法一次性加载文件中的所有数据到内存中，这个时候，我们就要利用外部排序算法。

大文件排序（外部排序）：如果文件过大，不能一次性加载到内存中，我们就需要使用外部排序算法，比如多路归并排序。简单来说，这个过程包括两步：首先，将大文件切分为多个小文件，并对每个小文件进行排序；然后，使用归并排序合并这些已经排序的小文件。这个过程需要一个自定义的归并算法，能够从每个小文件中读取一部分数据，然后进行排序。

一个常用的算法是“k路归并排序”。这个算法首先将大文件分割为多个小文件，然后对每个小文件进行排序，最后再将这些已经排序的小文件合并。下面是这个算法的步骤：

1. 将大文件分割为多个小文件。每个小文件的大小需要小于内存大小。
2. 对每个小文件进行排序，并将排序结果写入到新的小文件中。
3. 使用“k路归并排序”算法将这些小文件合并。这需要一个优先队列，优先队列中存储每个小文件当前的最小元素和对应的文件指针。每次从优先队列中取出最小元素，并将其写入到结果文件中，然后从对应的文件中读取一个新的元素放入到优先队列中。

以下是使用Python实现这个算法的代码：

```c#
import os
import heapq

def split_file(filename, temp_folder, chunk_size):
    smallfile = None
    with open(filename, 'r') as bigfile:
        for lineno, line in enumerate(bigfile):
            if lineno % chunk_size == 0:
                if smallfile:
                    smallfile.close()
                small_filename = os.path.join(temp_folder, 'small_file_{}.txt'.format(lineno // chunk_size))
                smallfile = open(small_filename, 'w')
            smallfile.write(line)
        if smallfile:
            smallfile.close()

def sort_small_files(temp_folder):
    small_files = os.listdir(temp_folder)
    for small_file in small_files:
        with open(os.path.join(temp_folder, small_file), 'r+') as f:
            numbers = list(map(int, f.read().strip().split(',')))
            numbers.sort()
            f.seek(0)
            f.write(','.join(map(str, numbers)))

def merge_files(temp_folder, output_file):
    small_files = [open(os.path.join(temp_folder, f)) for f in os.listdir(temp_folder)]
    queue = []
    for file_index, f in enumerate(small_files):
        line = f.readline().strip()
        if line:
            num = int(line)
            heapq.heappush(queue, (num, file_index))
    with open(output_file, 'w') as output_f:
        while queue:
            num, file_index = heapq.heappop(queue)
            output_f.write(str(num) + ',')
            line = small_files[file_index].readline().strip()
            if line:
                num = int(line)
                heapq.heappush(queue, (num, file_index))
    for f in small_files:
        f.close()

def sort_large_file(input_file, output_file, temp_folder='./temp', chunk_size=5000000):
    if not os.path.exists(temp_folder):
        os.makedirs(temp_folder)
    split_file(input_file, temp_folder, chunk_size)
    sort_small_files(temp_folder)
    merge_files(temp_folder, output_file)
    shutil.rmtree(temp_folder)

```

在C#中实现上述功能，需要使用到System.IO和System.Collections.Generic等命名空间。在C#中，我们可以使用StreamReader和StreamWriter来读写文件，使用List<T>和SortedSet<T>来实现优先队列。以下是一个简单的C#实现：

这段代码首先将大文件分割成小文件，并对小文件进行排序，然后再合并小文件。合并使用了优先队列SortedSet，这个数据结构能够在O(logN)的时间复杂度内找到并删除最小元素，和Python的heapq模块类似。

```c#
using System;
using System.IO;
using System.Collections.Generic;

class Program
{
    static int chunkSize = 1000000; // Modify this based on available memory

    static void Main(string[] args)
    {
        string sourceFile = "source.txt"; 
        string destinationFile = "destination.txt";
        string tempDirectory = "temp";
        ExternalSort(sourceFile, destinationFile, tempDirectory);
    }

    static void ExternalSort(string inputFile, string outputFile, string tempFolder)
    {
        Directory.CreateDirectory(tempFolder);

        var tempFiles = SplitAndSort(inputFile, tempFolder);
        Merge(tempFiles, outputFile);

        Directory.Delete(tempFolder, true);
    }

    static List<string> SplitAndSort(string inputFile, string tempFolder)
    {
        var tempFiles = new List<string>();
        using (var sr = new StreamReader(inputFile))
        {
            while (!sr.EndOfStream)
            {
                var nums = new List<int>();
                for (int i = 0; i < chunkSize && !sr.EndOfStream; i++)
                {
                    nums.Add(int.Parse(sr.ReadLine()));
                }
                nums.Sort();

                var tempFile = Path.Combine(tempFolder, Guid.NewGuid().ToString());
                using (var sw = new StreamWriter(tempFile))
                {
                    foreach (var num in nums)
                    {
                        sw.WriteLine(num);
                    }
                }

                tempFiles.Add(tempFile);
            }
        }
        return tempFiles;
    }

    static void Merge(List<string> tempFiles, string outputFile)
    {
        var queue = new SortedSet<Tuple<int, StreamReader>>(Comparer<Tuple<int, StreamReader>>.Create((x, y) => x.Item1 != y.Item1 ? x.Item1.CompareTo(y.Item1) : x.GetHashCode().CompareTo(y.GetHashCode())));
        var readers = new List<StreamReader>();
        try
        {
            foreach (var tempFile in tempFiles)
            {
                var reader = new StreamReader(tempFile);
                readers.Add(reader);
                if (!reader.EndOfStream)
                {
                    queue.Add(Tuple.Create(int.Parse(reader.ReadLine()), reader));
                }
            }

            using (var sw = new StreamWriter(outputFile))
            {
                while (queue.Count > 0)
                {
                    var tuple = queue.Min;
                    queue.Remove(tuple);

                    sw.WriteLine(tuple.Item1);

                    if (!tuple.Item2.EndOfStream)
                    {
                        queue.Add(Tuple.Create(int.Parse(tuple.Item2.ReadLine()), tuple.Item2));
                    }
                }
            }
        }
        finally
        {
            foreach (var reader in readers)
            {
                reader.Dispose();
            }
        }
    }
}

```

在`Merge`函数中，我们主要做的是将那些已经排好序的小文件合并成一个大文件。为了达到这个目标，我们需要在所有小文件的当前行（这些行都是文件中最小的未被写入大文件的行）中找出最小的，写入大文件，然后将那个小文件的指针向前移动一行。

下面我将详细解释一下`Merge`函数中的代码：

```csharp
var queue = new SortedSet<Tuple<int, StreamReader>>(Comparer<Tuple<int, StreamReader>>.Create((x, y) => x.Item1 != y.Item1 ? x.Item1.CompareTo(y.Item1) : x.GetHashCode().CompareTo(y.GetHashCode())));
```

上面这行代码创建了一个`SortedSet`对象，这个对象存储的是`Tuple<int, StreamReader>`对象。这个元组对象包含一个整数和一个`StreamReader`对象，整数代表当前行的值，`StreamReader`对象用于读取文件。这个`SortedSet`对象用于存储每个小文件当前最小的未被写入大文件的行，所以我们可以看到`SortedSet`对象中元组的第一个元素是当前行的值，第二个元素是对应文件的`StreamReader`对象。

这行代码初始化一个SortedSet。每个元素包含一个整数（文件中的一个数字）和一个StreamReader（对应的文件指针）。SortedSet会根据元素的第一部分（整数）进行排序。如果两个元素的第一部分相同，那么就比较它们的哈希码来决定顺序。这是因为SortedSet不允许插入重复元素，所以需要一种方法来确保即使两个元素的整数部分相同，它们也会被认为是不同的元素。


```csharp
foreach (var tempFile in tempFiles)
{
    var reader = new StreamReader(tempFile);
    readers.Add(reader);
    if (!reader.EndOfStream)
    {
        queue.Add(Tuple.Create(int.Parse(reader.ReadLine()), reader));
    }
}
```

在上面的代码中，我们遍历了每一个小文件，为每一个小文件创建了一个`StreamReader`对象，然后将这个`StreamReader`对象加入到了`readers`列表中。接着，如果文件没有结束（也就是文件中还有内容没有读取），我们就读取文件的当前行，并将这个行的值和对应的`StreamReader`对象一起加入到`queue`中。

这些行打开每一个临时文件，然后从文件中读取一个数字，并将这个数字和对应的文件指针作为一个元组添加到优先队列中。

```csharp
using (var sw = new StreamWriter(outputFile))
{
    while (queue.Count > 0)
    {
        var tuple = queue.Min;
        queue.Remove(tuple);

        sw.WriteLine(tuple.Item1);

        if (!tuple.Item2.EndOfStream)
        {
            queue.Add(Tuple.Create(int.Parse(tuple.Item2.ReadLine()), tuple.Item2));
        }
    }
}
```

在上面的代码中，我们创建了一个`StreamWriter`对象用于写入大文件。然后，只要`queue`中还有元素，我们就取出`queue`中最小的元素，将其写入到大文件中，然后将这个元素从队列中删除，从元组的第二个元素对应的文件中读取一个新的数字，然后将这个数字和对应的文件指针作为一个元组添加到优先队列中。



外部排序是指当数据量太大，无法一次性加载到内存中进行排序时，需要借助磁盘等外部存储介质进行排序的一类排序算法。这类算法主要面向大数据量的排序，核心思想是“分而治之”，即将大文件分割为多个小文件，然后在内存中对每个小文件进行排序，最后再将这些排序后的小文件合并。

多路归并排序是一种常见的外部排序算法，但不是唯一的。外部排序算法还有如下几种：

1. 两阶段多路合并排序：这是最基本的外部排序算法，它首先读取一个数据块到内存，进行内部排序后输出，重复这个过程直到所有数据块都排序完，然后再将这些排序后的数据块进行多路归并。多路归并排序就是这种算法的一种。

2. 替换选择排序：这是一种改进的外部排序算法，它在第一阶段产生排序后的数据块时，能产生比内存大的数据块，从而减少了最后合并的次数。

3. 聚合排序（Polyphase merge sort）：这种方法尝试通过一次分配将原始数据的许多阶段集中在一起，减少了读写磁盘的次数。

4. 层叠排序（Cascade merge sort）：类似于多路归并排序，但使用了更复杂的调度算法，可以减少磁盘的读写次数。

这些算法都没有成熟的框架可以直接使用，需要根据具体的情况进行编程实现。然而，对于非常大的数据量，如TB级别的数据，通常会使用分布式计算框架如Hadoop和Spark来进行排序。这些框架的底层实现了MapReduce编程模型，其中的Reduce阶段就是进行排序和归并操作。这样就可以将数据分布在多台计算机上进行处理，大大提高了排序的效率。


3. 多核排序：如果要利用多核CPU，我们需要加入并发或者并行的操作。在Python中，我们可以使用多线程或者多进程来实现。具体来说，我们可以创建多个线程或进程，每个线程或进程处理一个小文件的排序。然后，再用一个线程或进程来处理归并排序。

首先我们需要理解Python的多线程和多进程模型。Python的多线程由于GIL(Global Interpreter Lock)的存在，多线程并不能充分利用多核优势，多线程主要用在I/O密集型任务而不是计算密集型任务。如果你的任务主要是计算密集型任务，你应该使用多进程而不是多线程。

我们可以使用Python的concurrent.futures模块，它为我们提供了ProcessPoolExecutor类，这个类可以帮助我们在一个进程池中运行多个进程。然后我们可以使用进程池来并行处理多个小文件的排序。这里还需要一个归并排序的方法来合并这些已经排序的小文件。

以下是基本的思路和代码：

1. 首先我们需要把大文件分成多个小文件，并对每个小文件进行排序。我们可以创建一个ProcessPoolExecutor，然后把每个小文件的排序任务提交到进程池中。

```python
import concurrent.futures

def sort_small_file(filename):
    with open(filename, 'r') as file:
        numbers = list(map(int, file.read().split(',')))
    numbers.sort()
    with open(filename, 'w') as file:
        file.write(','.join(map(str, numbers)))

def split_and_sort_large_file(large_filename, small_filename_prefix, num_files):
    # split the large file into small files here
    # then submit the sort task to the process pool
    with concurrent.futures.ProcessPoolExecutor() as executor:
        future_to_file = {executor.submit(sort_small_file, f"{small_filename_prefix}_{i}"): i for i in range(num_files)}
        for future in concurrent.futures.as_completed(future_to_file):
            file_index = future_to_file[future]
            try:
                future.result()
                print(f"File {file_index} sorted successfully")
            except Exception as exc:
                print(f"File {file_index} generated an exception: {exc}")

```

然后我们需要对这些已经排序的小文件进行合并。我们可以使用一个优先级队列来做这个事情，优先级队列可以高效地找到最小的元素。

```python
import heapq
from typing import List

def merge_sorted_files(sorted_filenames: List[str], output_filename):
    sorted_files = [open(filename, 'r') for filename in sorted_filenames]
    heap = [(int(line.split(',')[0]), i) for i, line in enumerate(file.readline() for file in sorted_files)]
    heapq.heapify(heap)
    with open(output_filename, 'w') as output_file:
        while heap:
            smallest_entry, smallest_file_index = heapq.heappop(heap)
            output_file.write(str(smallest_entry))
            next_line = sorted_files[smallest_file_index].readline()
            if next_line:
                heapq.heappush(heap, (int(next_line.split(',')[0]), smallest_file_index))
    for file in sorted_files:
        file.close()
```

这里的代码只是一个基本的示例，实际的实现可能会更复杂，需要处理一些边界情况和错误情况。此外，这里我们假设每个小文件都可以装进内存，如果这个假设不成立，你可能需要使用一种更复杂的归并算法来处理这个问题。


可以使用C#来实现利用多核CPU进行排序的功能。C#的Task Parallel Library (TPL) 提供了强大的并发计算功能。

以下是基本的思路和代码：

1. 首先我们需要把大文件分成多个小文件，并对每个小文件进行排序。我们可以创建一个任务列表，并把每个小文件的排序任务提交到任务列表中。

```c#
using System;
using System.IO;
using System.Linq;
using System.Threading.Tasks;

public class MultiCoreSorter
{
    public void SortSmallFile(string fileName)
    {
        var numbers = File.ReadAllText(fileName)
            .Split(',')
            .Select(int.Parse)
            .OrderBy(x => x)
            .ToArray();

        File.WriteAllText(fileName, string.Join(",", numbers));
    }

    public async Task SplitAndSortLargeFile(string largeFileName, string smallFileNamePrefix, int numFiles)
    {
        // Assume that you have a method to split the large file into small files
        // and the name of each small file is $"{smallFileNamePrefix}_{i}"

        var tasks = new Task[numFiles];
        for (int i = 0; i < numFiles; i++)
        {
            string smallFileName = $"{smallFileNamePrefix}_{i}";
            tasks[i] = Task.Run(() => SortSmallFile(smallFileName));
        }
        await Task.WhenAll(tasks);
    }
}

```

2. 然后我们需要对这些已经排序的小文件进行合并。我们可以使用一个优先队列来做这个事情。这一部分的逻辑跟Python的归并有点复杂，你需要一个优先队列数据结构，C#的标准库里并没有提供，所以你可能需要自己实现，或者使用第三方库。

```c#
public async Task MergeSortedFiles(string[] sortedFileNames, string outputFileName)
{
    var readers = sortedFileNames.Select(fn => new StreamReader(fn)).ToArray();
    var queue = new SortedList<int, StreamReader>();

    for (int i = 0; i < readers.Length; i++)
    {
        var line = await readers[i].ReadLineAsync();
        if (line != null)
        {
            var value = int.Parse(line.Split(',')[0]);
            queue.Add(value, readers[i]);
        }
    }

    using var writer = new StreamWriter(outputFileName);
    while (queue.Count > 0)
    {
        var minReader = queue.Values[0];
        var minValue = queue.Keys[0];
        queue.RemoveAt(0);
        await writer.WriteAsync(minValue + ",");

        var line = await minReader.ReadLineAsync();
        if (line != null)
        {
            var nextValue = int.Parse(line.Split(',')[0]);
            queue.Add(nextValue, minReader);
        }
    }

    foreach (var reader in readers)
    {
        reader.Close();
    }
}

```

这段代码只是一个基本的示例，它没有处理错误和异常。在实际应用中，你可能需要增加错误处理代码，并优化性能。

对于.NET平台，`Task`基于线程，而非进程。但是.NET并不受Python GIL（全局解释器锁）限制的影响，所以在.NET中，可以有效地利用多线程来提高计算密集型任务的效率。这是因为.NET Framework和.NET Core的并发和并行编程模型比Python的模型更成熟，它们可以在多线程和任务并行库（TPL）中更有效地利用多核处理器。

所以，在C#中，我们可以使用`Task`来处理计算密集型任务，因为.NET运行时可以有效地在多个线程之间分配任务，并利用多核处理器。即使你的任务主要是计算密集型的，使用`Task`依然是合适的。

但是需要注意的是，如果你的任务需要大量的CPU计算并且任务之间没有太多的依赖关系，使用多进程可能会获得更好的性能。因为进程之间的隔离更好，每个进程都有自己的内存空间，而线程之间共享同一块内存空间，因此在多线程编程中需要更小心地处理数据共享和同步问题。

总的来说，选择使用`Task`还是使用多进程，主要取决于你的具体需求和你的应用场景。在大多数情况下，使用`Task`应该是足够的，并且更容易编程。

4. 分布式排序（MapReduce）：如果文件非常大，我们需要使用分布式系统来进行排序。这通常需要一个分布式计算框架，比如Hadoop或者Spark。在这个过程中，Map阶段将大文件切分为多个小文件，并对每个小文件进行排序；Reduce阶段则负责合并这些已经排序的小文件。这个过程需要专业的分布式计算知识，并且需要对Hadoop或者Spark等工具有一定的理解。

使用 Hadoop MapReduce 进行大规模数据排序的大致思路如下：

1. Map阶段：Map 阶段读取输入的文件，并将每一行（或者每一个数值）作为一个输入记录。然后，对这些记录按值进行排序。Hadoop 默认情况下会对 Map 的输出进行排序，所以在这个阶段我们不需要做太多的工作。

2. Reduce阶段：在 Reduce 阶段，Hadoop 会把所有的相同键值的数据送到同一个 Reducer 进行处理。然后每个 Reducer 将数据输出到磁盘，这就是最后排序好的结果。


在Hadoop MapReduce中进行分布式排序通常包括两个步骤：Map阶段和Reduce阶段。以下是这两个步骤的基本工作原理：

1. Map阶段：将大文件分割为许多小的分块，并对每个分块的数据进行排序。
2. Reduce阶段：接收所有Map阶段输出的排序结果，然后进行归并排序，最后输出排序后的整体结果。
这里是一个简单的Java实现（因为Hadoop的原生支持语言是Java）：

```java
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class Sort {

    // Map阶段
    public static class SortMapper extends Mapper<LongWritable, Text, IntWritable, IntWritable> {
        private IntWritable data = new IntWritable();

        public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            String line = value.toString();
            String[] numbers = line.split(",");
            for (String num : numbers) {
                data.set(Integer.parseInt(num));
                context.write(data, new IntWritable(1));
            }
        }
    }

    // Reduce阶段
    public static class SortReducer extends Reducer<IntWritable, IntWritable, IntWritable, IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(IntWritable key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
            for (IntWritable val : values) {
                context.write(key, val);
            }
        }
    }

    // Job配置
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "sort");
        job.setJarByClass(Sort.class);
        job.setMapperClass(SortMapper.class);
        job.setReducerClass(SortReducer.class);
        job.setOutputKeyClass(IntWritable.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}

```

这段Java代码实现了一个MapReduce排序程序。它首先定义了一个Mapper类和一个Reducer类，然后在主函数中设置了作业配置，指定了Mapper和Reducer类，输入输出的路径。

在Map阶段，每行数据被切分并转换为整数，然后被发送到Reduce阶段。

在Reduce阶段，所有的输入键值对按键进行排序，然后输出。

需要注意的是，你需要一个运行着Hadoop的集群，并且正确地配置了Hadoop的环境。另外，这段代码并未处理任何错误或异常，你可能需要根据你的具体需求来增加错误处理和数据验证。


这里我们选择使用Spark来处理这个问题，因为Spark比Hadoop更快，API更易于使用。

下面是一个在Spark中实现的基本排序算法。首先，我们需要设置Spark环境，然后加载数据，最后进行排序。

请注意，这是一个非常基础的实现，没有处理数据清洗、异常处理等一些可能的问题。实际生产环境中，还需要根据具体情况进行优化。

```python
from pyspark import SparkConf, SparkContext

def sort_large_file():
    # 创建Spark配置
    conf = SparkConf().setAppName("SortLargeFile")
    sc = SparkContext(conf=conf)

    # 加载数据
    data = sc.textFile("hdfs://localhost:9000/path/to/your/file")

    # 将数据切割并转换为整数
    numbers = data.flatMap(lambda line: line.split(",")).map(lambda num: int(num))

    # 对数据进行排序
    sorted_numbers = numbers.sortBy(lambda num: num)

    # 将结果保存回HDFS
    sorted_numbers.saveAsTextFile("hdfs://localhost:9000/path/to/output")

    # 停止SparkContext
    sc.stop()

```

此代码首先配置了一个Spark应用，然后从HDFS中读取了一个大文件。这个文件被切分为许多小的分区，每个分区都在Spark集群的不同节点上进行处理。

在flatMap阶段，数据被切割，并转换为整数。然后，数据被排序，并保存回HDFS。

需要注意的是，你需要有一个运行着Spark和HDFS的集群，并且正确配置了Spark和HDFS的环境。另外，这段代码没有处理任何错误或异常，你可能需要根据你的具体需求来增加错误处理和数据验证。


## Design Patterns 笔记

Composition类维护一个Component实例的集合，它们代表文档中的文本和图形元素。Composition使用一个Compositor子类的实例，将组件对象排列成行，这个子类封装了一个换行策略。每个组件都有一个关联的自然大小、可伸展性和可收缩性。可伸展性定义了组件可以超出其自然大小的程度；可收缩性则定义了它可以缩小的程度。Composition将这些值传递给Compositor，Compositor利用这些值确定换行的最佳位置。

当需要新的布局时，Composition会要求其Compositor确定放置换行符的位置。Composition会向Compositor传递三个数组，定义组件的自然大小、可伸展性和可收缩性。它还传递组件的数量，行的宽度，以及一个Compositor用于填充每个换行符位置的数组。Compositor返回计算的断行数。


Compositor接口允许Composition将所有信息传递给Compositor。这是"将数据带入策略"的一个例子。需要注意的是，Compositor是一个抽象类。具体的子类定义了特定的换行策略。


要实例化Composition，你需要传递你想要使用的Compositor

Compositor的接口设计得非常仔细，以支持所有子类可能实现的布局算法。你不希望因为每一个新的子类而改变这个接口，因为这将需要改变现有的子类。总的来说，策略和上下文接口决定了模式如何实现其意图。


```csharp
public interface ICompositor
{
    int Compose(int[] natural, int[] stretch, int[] shrink, int componentCount, int lineWidth, int[] breaks);
}

public class Composition
{
    private ICompositor _compositor;
    private int[] _components; // 组件列表
    private int _componentCount; // 组件数量
    private int _lineWidth; // Composition的行宽
    private int[] _lineBreaks; // 组件中的断行位置
    private int _lineCount; // 行数量

    public Composition(ICompositor compositor)
    {
        this._compositor = compositor;
    }

    public void Repair()
    {
        int[] natural = new int[_componentCount];
        int[] stretchability = new int[_componentCount];
        int[] shrinkability = new int[_componentCount];
        int[] breaks = new int[_componentCount];

        // 准备具有所需组件大小的数组
        // ...
        
        // 确定断行位置
        int breakCount = _compositor.Compose(natural, stretchability, shrinkability, _componentCount, _lineWidth, breaks);

        // 根据断行布局组件
        // ...
    }
}

public class SimpleCompositor : ICompositor
{
    public int Compose(int[] natural, int[] stretch, int[] shrink, int componentCount, int lineWidth, int[] breaks)
    {
        // 实现SimpleCompositor的Compose方法
        // ...
        return 0;
    }
}

public class TeXCompositor : ICompositor
{
    public int Compose(int[] natural, int[] stretch, int[] shrink, int componentCount, int lineWidth, int[] breaks)
    {
        // 实现TeXCompositor的Compose方法
        // ...
        return 0;
    }
}

public class ArrayCompositor : ICompositor
{
    private int _interval;

    public ArrayCompositor(int interval)
    {
        _interval = interval;
    }

    public int Compose(int[] natural, int[] stretch, int[] shrink, int componentCount, int lineWidth, int[] breaks)
    {
        // 实现ArrayCompositor的Compose方法
        // ...
        return 0;
    }
}

```

```csharp
Composition quick = new Composition(new SimpleCompositor());
Composition slick = new Composition(new TeXCompositor());
Composition iconic = new Composition(new ArrayCompositor(100));
```

## agile patterns

## .net framework Compare

在.NET中，IComparer<T>和Comparison<T>接口提供了一种定义自定义排序策略的方式。它们通常用于List<T>.Sort方法或Array.Sort方法中，以定义如何比较类型T的两个实例。

首先，我们来看看IComparer<T>接口。这个接口定义了一个Compare方法，这个方法接受两个类型T的参数，并返回一个表示这两个对象相对排序的int值。

```c#
namespace System.Collections.Generic {
    
    using System;
    // The generic IComparer interface implements a method that compares 
    // two objects. It is used in conjunction with the Sort and 
    // BinarySearch methods on the Array, List, and SortedList classes.
    public interface IComparer<in T>
    {
        // Compares two objects. An implementation of this method must return a
        // value less than zero if x is less than y, zero if x is equal to y, or a
        // value greater than zero if x is greater than y.
        // 
        int Compare(T x, T y);
    }
}
```
以下是一个使用IComparer<T>接口来定义自定义排序策略的示例：

```c#
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
}

public class PersonComparer : IComparer<Person>
{
    public int Compare(Person x, Person y)
    {
        // 先按姓氏排序，如果姓氏相同，再按名字排序
        int result = string.Compare(x.LastName, y.LastName);
        if (result == 0)
        {
            result = string.Compare(x.FirstName, y.FirstName);
        }
        return result;
    }
}

// 使用示例
List<Person> people = new List<Person>
{
    new Person { FirstName = "John", LastName = "Doe" },
    new Person { FirstName = "Jane", LastName = "Doe" },
    new Person { FirstName = "John", LastName = "Smith" }
};

people.Sort(new PersonComparer());

foreach (var person in people)
{
    Console.WriteLine($"{person.LastName}, {person.FirstName}");
}

```

在上面的例子中，我们创建了一个名为 PersonComparer 的类，它实现了IComparer<Person> 接口。我们在 Compare 方法中定义了一个比较两个Person 对象的策略：首先按照姓（LastName）进行排序，如果姓相同，则按照名（FirstName）进行排序。

```c#
// Sorts the elements in this list.  Uses the default comparer and 
        // Array.Sort.
        public void Sort()
        {
            Sort(0, Count, null);
        }
 
        // Sorts the elements in this list.  Uses Array.Sort with the
        // provided comparer.
        public void Sort(IComparer<T> comparer)
        {
            Sort(0, Count, comparer);
        }

                // Sorts the elements in a section of this list. The sort compares the
        // elements to each other using the given IComparer interface. If
        // comparer is null, the elements are compared to each other using
        // the IComparable interface, which in that case must be implemented by all
        // elements of the list.
        // 
        // This method uses the Array.Sort method to sort the elements.
        // 
        public void Sort(int index, int count, IComparer<T> comparer) {
            if (index < 0) {
                ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.index, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
            }
            
            if (count < 0) {
                ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.count, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
            }
                
            if (_size - index < count)
                ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_InvalidOffLen);
            Contract.EndContractBlock();
 
            Array.Sort<T>(_items, index, count, comparer);
            _version++;
        }
```

```c#
        [System.Security.SecuritySafeCritical]  // auto-generated
        [ReliabilityContract(Consistency.MayCorruptInstance, Cer.MayFail)]
        public static void Sort<T>(T[] array, int index, int length, System.Collections.Generic.IComparer<T> comparer) {
            if (array==null)
                throw new ArgumentNullException("array");
            if (index < 0 || length < 0)
                throw new ArgumentOutOfRangeException((length<0 ? "length" : "index"), Environment.GetResourceString("ArgumentOutOfRange_NeedNonNegNum"));
            if (array.Length - index < length)
                throw new ArgumentException(Environment.GetResourceString("Argument_InvalidOffLen"));
            Contract.EndContractBlock();
 
            if (length > 1) {
                // <
 
 
 
                if ( comparer == null || comparer == Comparer<T>.Default ) {
                    if(TrySZSort(array, null, index, index + length - 1)) {
                        return;
                    }
                }
                
#if FEATURE_LEGACYNETCF
                if (CompatibilitySwitches.IsAppEarlierThanWindowsPhone8)
                    MangoArraySortHelper<T>.Default.Sort(array, index, length, comparer);                
                else
                    ArraySortHelper<T>.Default.Sort(array, index, length, comparer);                
#else
                ArraySortHelper<T>.Default.Sort(array, index, length, comparer);                
#endif
            }
        }
```

```c#
 public void Sort(T[] keys, int index, int length, IComparer<T> comparer)
        {
            Contract.Assert(keys != null, "Check the arguments in the caller!");
            Contract.Assert( index >= 0 && length >= 0 && (keys.Length - index >= length), "Check the arguments in the caller!");
 
            // Add a try block here to detect IComparers (or their
            // underlying IComparables, etc) that are bogus.
            try
            {
                if (comparer == null)
                {
                    comparer = Comparer<T>.Default;
                }
 
#if FEATURE_CORECLR
                // Since QuickSort and IntrospectiveSort produce different sorting sequence for equal keys the upgrade 
                // to IntrospectiveSort was quirked. However since the phone builds always shipped with the new sort aka 
                // IntrospectiveSort and we would want to continue using this sort moving forward CoreCLR always uses the new sort.
 
                IntrospectiveSort(keys, index, length, comparer);
#else
                if (BinaryCompatibility.TargetsAtLeast_Desktop_V4_5)
                {
                    IntrospectiveSort(keys, index, length, comparer);
                }
                else
                {
                    DepthLimitedQuickSort(keys, index, length + index - 1, comparer, IntrospectiveSortUtilities.QuickSortDepthThreshold);
                }
#endif
            }
            catch (IndexOutOfRangeException)
            {
                IntrospectiveSortUtilities.ThrowOrIgnoreBadComparer(comparer);
            }
            catch (Exception e)
            {
                throw new InvalidOperationException(Environment.GetResourceString("InvalidOperation_IComparerFailed"), e);
            }
        }
```

```c#
        internal static void IntrospectiveSort(T[] keys, int left, int length, IComparer<T> comparer)
        {
            Contract.Requires(keys != null);
            Contract.Requires(comparer != null);
            Contract.Requires(left >= 0);
            Contract.Requires(length >= 0);
            Contract.Requires(length <= keys.Length);
            Contract.Requires(length + left <= keys.Length);
 
            if (length < 2)
                return;
 
            IntroSort(keys, left, length + left - 1, 2 * IntrospectiveSortUtilities.FloorLog2(keys.Length), comparer);
        }
```

```c#
 private static void IntroSort(T[] keys, int lo, int hi, int depthLimit, IComparer<T> comparer)
        {
            Contract.Requires(keys != null);
            Contract.Requires(comparer != null);
            Contract.Requires(lo >= 0);
            Contract.Requires(hi < keys.Length);
 
            while (hi > lo)
            {
                int partitionSize = hi - lo + 1;
                if (partitionSize <= IntrospectiveSortUtilities.IntrosortSizeThreshold)
                {
                    if (partitionSize == 1)
                    {
                        return;
                    }
                    if (partitionSize == 2)
                    {
                        SwapIfGreater(keys, comparer, lo, hi);
                        return;
                    }
                    if (partitionSize == 3)
                    {
                        SwapIfGreater(keys, comparer, lo, hi-1);
                        SwapIfGreater(keys, comparer, lo, hi);
                        SwapIfGreater(keys, comparer, hi-1, hi);
                        return;
                    }
 
                    InsertionSort(keys, lo, hi, comparer);
                    return;
                }
 
                if (depthLimit == 0)
                {
                    Heapsort(keys, lo, hi, comparer);
                    return;
                }
                depthLimit--;
 
                int p = PickPivotAndPartition(keys, lo, hi, comparer);
                // Note we've already partitioned around the pivot and do not have to move the pivot again.
                IntroSort(keys, p + 1, hi, depthLimit, comparer);
                hi = p - 1;
            }
        }
```

```c#
private static void SwapIfGreater(T[] keys, IComparer<T> comparer, int a, int b)
        {
            if (a != b)
            {
                if (comparer.Compare(keys[a], keys[b]) > 0)
                {
                    T key = keys[a];
                    keys[a] = keys[b];
                    keys[b] = key;
                }
            }
        }
 
```

```c#
private static void InsertionSort(T[] keys, int lo, int hi, IComparer<T> comparer)
        {
            Contract.Requires(keys != null);
            Contract.Requires(lo >= 0);
            Contract.Requires(hi >= lo);
            Contract.Requires(hi <= keys.Length);
 
            int i, j;
            T t;
            for (i = lo; i < hi; i++)
            {
                j = i;
                t = keys[i + 1];
                while (j >= lo && comparer.Compare(t, keys[j]) < 0)
                {
                    keys[j + 1] = keys[j];
                    j--;
                }
                keys[j + 1] = t;
            }
        }
```

```c#
private static void Heapsort(T[] keys, int lo, int hi, IComparer<T> comparer)
        {
            Contract.Requires(keys != null);
            Contract.Requires(comparer != null);
            Contract.Requires(lo >= 0);
            Contract.Requires(hi > lo);
            Contract.Requires(hi < keys.Length);
 
            int n = hi - lo + 1;
            for (int i = n / 2; i >= 1; i = i - 1)
            {
                DownHeap(keys, i, n, lo, comparer);
            }
            for (int i = n; i > 1; i = i - 1)
            {
                Swap(keys, lo, lo + i - 1);
                DownHeap(keys, 1, i - 1, lo, comparer);
            }
        }
```

```c#
private static void DownHeap(T[] keys, int i, int n, int lo, IComparer<T> comparer)
        {
            Contract.Requires(keys != null);
            Contract.Requires(comparer != null);
            Contract.Requires(lo >= 0);
            Contract.Requires(lo < keys.Length);
 
            T d = keys[lo + i - 1];
            int child;
            while (i <= n / 2)
            {
                child = 2 * i;
                if (child < n && comparer.Compare(keys[lo + child - 1], keys[lo + child]) < 0)
                {
                    child++;
                }
                if (!(comparer.Compare(d, keys[lo + child - 1]) < 0))
                    break;
                keys[lo + i - 1] = keys[lo + child - 1];
                i = child;
            }
            keys[lo + i - 1] = d;
        }
```

接下来，我们来看看Comparison<T>。Comparison<T>是一个委托，它表示将两个类型T的对象作为输入，并返回一个表示它们相对排序的int的方法。在许多情况下，使用Comparison<T>可以使代码更简洁，因为你可以使用Lambda表达式或方法组来表示比较逻辑，而不是创建一个实现IComparer<T>的单独类。

```c#
public delegate int Comparison<in T>(T x, T y);
```

以下是一个使用Comparison<T>的示例：

```c#
List<Person> people = new List<Person>
{
    new Person { FirstName = "John", LastName = "Doe" },
    new Person { FirstName = "Jane", LastName = "Doe" },
    new Person { FirstName = "John", LastName = "Smith" }
};

Comparison<Person> personComparison = (x, y) =>
{
    int result = string.Compare(x.LastName, y.LastName);
    if (result == 0)
    {
        result = string.Compare(x.FirstName, y.FirstName);
    }
    return result;
};

people.Sort(personComparison);

foreach (var person in people)
{
    Console.WriteLine($"{person.LastName}, {person.FirstName}");
}

```

在这个例子中，我们使用Lambda表达式定义了一个Comparison<Person>。这个Lambda表达式表示的函数与前面例子中PersonComparer类的Compare方法相同。然后，我们将这个Comparison传递给List<Person>.Sort方法来对人员列表进行排序。

```c#
public void Sort(Comparison<T> comparison) {
            if( comparison == null) {
                ThrowHelper.ThrowArgumentNullException(ExceptionArgument.match);
            }
            Contract.EndContractBlock();
 
            if( _size > 0) {
                IComparer<T> comparer = new Array.FunctorComparer<T>(comparison);
                Array.Sort(_items, 0, _size, comparer);
            }
        }
```

```c#
internal sealed class FunctorComparer<T> : IComparer<T> {
            Comparison<T> comparison;
 
            public FunctorComparer(Comparison<T> comparison) {
                this.comparison = comparison;
            }
 
            public int Compare(T x, T y) {
                return comparison(x, y);
            }
        }
```