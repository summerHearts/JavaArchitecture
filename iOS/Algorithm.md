## 1、字符逆序算法

```
@interface CharReverse : NSObject
// 字符串反转
void char_reverse(char* cha);
@end

@implementation CharReverse

void char_reverse(char* cha){
    // 指向第一个字符
    char* begin = cha;
    // 指向最后一个字符
    char* end = cha + strlen(cha) - 1;
    
    while (begin < end) {
        // 交换前后两个字符,同时移动指针
        char temp = *begin;
        *(begin++) = *end;
        *(end--) = temp;
    }
}

@end
```

## 2、 链表反转算法

```
// 定义一个链表
struct Node {
    int data;
    struct Node *next;
};

@interface ReverseList : NSObject
// 链表反转
struct Node* reverseList(struct Node *head);
// 构造一个链表
struct Node* constructList(void);
// 打印链表中的数据
void printList(struct Node *head);

@end

@implementation ReverseList

struct Node* reverseList(struct Node *head)
{
    // 定义遍历指针，初始化为头结点
    struct Node *p = head;
    // 反转后的链表头部
    struct Node *newH = NULL;
    
    // 遍历链表
    while (p != NULL) {
        
        // 记录下一个结点
        struct Node *temp = p->next;
        // 当前结点的next指向新链表头部
        p->next = newH;
        // 更改新链表头部为当前结点
        newH = p;
        // 移动p指针
        p = temp;
    }
    
    // 返回反转后的链表头结点
    return newH;
}

struct Node* constructList(void)
{
    // 头结点定义
    struct Node *head = NULL;
    // 记录当前尾结点
    struct Node *cur = NULL;
    
    for (int i = 1; i < 5; i++) {
        struct Node *node = malloc(sizeof(struct Node));
        node->data = i;
        node->next = NULL;
        
        // 头结点为空，新结点即为头结点
        if (head == NULL) {
            head = node;
        }
        // 当前结点的next为新结点
        else{
            cur->next = node;
        }
        
        // 设置当前结点为新结点
        cur = node;
    }
    
    return head;
}

void printList(struct Node *head)
{
    struct Node* temp = head;
    while (temp != NULL) {
        printf("node is %d \n", temp->data);
        temp = temp->next;
    }
}

@end
```

## 3、有序数组合并算法

```
@interface MergeSortedList : NSObject
// 将有序数组a和b的值合并到一个数组result当中，且仍然保持有序
void mergeList(int a[], int aLen, int b[], int bLen, int result[]);

@end

@implementation MergeSortedList

void mergeList(int a[], int aLen, int b[], int bLen, int result[])
{
    int p = 0; // 遍历数组a的指针
    int q = 0; // 遍历数组b的指针
    int i = 0; // 记录当前存储位置
    
    // 任一数组没有到达边界则进行遍历
    while (p < aLen && q < bLen) {
        // 如果a数组对应位置的值小于b数组对应位置的值
        if (a[p] <= b[q]) {
            // 存储a数组的值
            result[i] = a[p];
            // 移动a数组的遍历指针
            p++;
        }
        else{
            // 存储b数组的值
            result[i] = b[q];
            // 移动b数组的遍历指针
            q++;
        }
        // 指向合并结果的下一个存储位置
        i++;
    }
    
    // 如果a数组有剩余
    while (p < aLen) {
        // 将a数组剩余部分拼接到合并结果的后面
        result[i] = a[p++];
        i++;
    }
    
    // 如果b数组有剩余
    while (q < bLen) {
        // 将b数组剩余部分拼接到合并结果的后面
        result[i] = b[q++];
        i++;
    }
}

@end
```

## 4、Hash算法

```
@interface HashFind : NSObject

// 查找第一个只出现一次的字符
char findFirstChar(char* cha);

@end

@implementation HashFind

char findFirstChar(char* cha){
    char result = '\0';
    // 定义一个数组 用来存储各个字母出现次数
    int array[256];
    // 对数组进行初始化操作
    for (int i=0; i<256; i++) {
        array[i] =0;
    }
    // 定义一个指针 指向当前字符串头部
    char* p = cha;
    // 遍历每个字符
    while (*p != '\0') {
        // 在字母对应存储位置 进行出现次数+1操作
        array[*(p++)]++;
    }
    
    // 将P指针重新指向字符串头部
    p = cha;
    // 遍历每个字母的出现次数
    while (*p != '\0') {
        // 遇到第一个出现次数为1的字符，打印结果
        if (array[*p] == 1)
        {
            result = *p;
            break;
        }
        // 反之继续向后遍历
        p++;
    }
    
    return result;
}

@end
```

## 5、查找两个子视图的共同父视图算法

```
@interface CommonSuperFind : NSObject

// 查找两个视图的共同父视图
- (NSArray<UIView *> *)findCommonSuperView:(UIView *)view other:(UIView *)viewOther;

@end

@implementation CommonSuperFind

- (NSArray <UIView *> *)findCommonSuperView:(UIView *)viewOne other:(UIView *)viewOther{
    NSMutableArray *result = [NSMutableArray array];
    
    // 查找第一个视图的所有父视图
    NSArray *arrayOne = [self findSuperViews:viewOne];
    // 查找第二个视图的所有父视图
    NSArray *arrayOther = [self findSuperViews:viewOther];
    
    int i = 0;
    // 越界限制条件
    while (i < MIN((int)arrayOne.count, (int)arrayOther.count)) {
        // 倒序方式获取各个视图的父视图
        UIView *superOne = [arrayOne objectAtIndex:arrayOne.count - i - 1];
        UIView *superOther = [arrayOther objectAtIndex:arrayOther.count - i - 1];
        
        // 比较如果相等 则为共同父视图
        if (superOne == superOther) {
            [result addObject:superOne];
            i++;
        }
        // 如果不相等，则结束遍历
        else{
            break;
        }
    }
    
    return result;
}

- (NSArray <UIView *> *)findSuperViews:(UIView *)view{
    // 初始化为第一父视图
    UIView *temp = view.superview;
    // 保存结果的数组
    NSMutableArray *result = [NSMutableArray array];
    while (temp) {
        [result addObject:temp];
        // 顺着superview指针一直向上查找
        temp = temp.superview;
    }
    return result;
}

@end
```
## 5、无序数组当中的中位数算法

```
@interface MedianFind : NSObject
// 无序数组中位数查找
int findMedian(int a[], int aLen);

@end

@implementation MedianFind

//求一个无序数组的中位数
int findMedian(int a[], int aLen)
{
    int low = 0;
    int high = aLen - 1;
    
    int mid = (aLen - 1) / 2;
    int div = PartSort(a, low, high);
    
    while (div != mid)
    {
        if (mid < div)
        {
            //左半区间找
            div = PartSort(a, low, div - 1);
        }
        else
        {
            //右半区间找
            div = PartSort(a, div + 1, high);
        }
    }
    //找到了
    return a[mid];
}

int PartSort(int a[], int start, int end)
{
    int low = start;
    int high = end;
    
    //选取关键字
    int key = a[end];
    
    while (low < high)
    {
        //左边找比key大的值
        while (low < high && a[low] <= key)
        {
            ++low;
        }
        
        //右边找比key小的值
        while (low < high && a[high] >= key)
        {
            --high;
        }
        
        if (low < high)
        {
            //找到之后交换左右的值
            int temp = a[low];
            a[low] = a[high];
            a[high] = temp;
        }
    }
    
    int temp = a[high];
    a[high] = a[end];
    a[end] = temp;
    
    return low;
}

@end
```

## 6、递归实现全排列

![](https://upload-images.jianshu.io/upload_images/325120-1f1bac3c424b3f47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

```
NSArray *sortalgorithm(NSString *strOfValue){
    //用于存储全排序列
    NSMutableArray *marrOfSort = [[NSMutableArray alloc] init];
    
    //若字符串为单个字符 直接返回单个字符 的数组
    if ([strOfValue length] == 0) {
        [marrOfSort addObject:strOfValue];
        return marrOfSort;
    }else{
        for (int i = 0; i < [strOfValue length]; i++) {
            //依次选择分割哪一个字符
            char OneOfChar = [strOfValue characterAtIndex:i];
            
            NSMutableString *strOfDeleteOneChars = [NSMutableString stringWithString:strOfValue];
            //分割字符串
            [strOfDeleteOneChars deleteCharactersInRange:NSMakeRange(i, 1)];
            NSArray *arrOfSub = sortalgorithm(strOfDeleteOneChars);/*递归*/
            for (int j = 0; j < [arrOfSub count]; j++) {
                NSString *strOfCombine = [NSString stringWithFormat:@"%c%@",OneOfChar,[arrOfSub objectAtIndex:j]];
                [marrOfSort addObject:strOfCombine];
                
            }
        }
        return marrOfSort;
        
    }
}
```

组合指从n个不同元素中取出m个元素来合成的一个组，这个组内元素没有顺序。使用C(n, k)表示从n个元素中取出k个元素的取法数。

如数组为{1, 2, 3, 4, 5, 6}，那么从它中取出3个元素的组合有哪些，取出4个元素的组合呢？
比如取3个元素的组合，我们的思维是：
取1、2，然后再分别取3，4，5，6；
取1、3，然后再分别取4，5，6；
......
取2、3，然后再分别取4，5，5；
......
这样按顺序来，就可以保证完全没有重复。

这种顺序思维给我们的启示便是这个问题可以用递归来实现，但是仅从上述描述来看，却无法下手。
我们可以稍作改变：
1.先从数组中A取出一个元素，然后再从余下的元素B中取出一个元素，然后又在余下的元素C中取出一个元素
2.按照数组索引从小到大依次取，避免重复

```
void combine_increase(int* arr, int start, int* result, int count, const int NUM, const int arr_len){
    int i = 0;
    for (i = start; i < arr_len + 1 - count; i++)
    {
        result[count - 1] = i;
        if (count - 1 == 0)
        {
            int j;
            for (j = NUM - 1; j >= 0; j--)
                printf("%d",arr[result[j]]);
            printf("\n");
        }
        else
            combine_increase(arr, i + 1, result, count - 1, NUM, arr_len);
    }
}



int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        NSLog(@"Hello, World!");
        
        
       int arr[] = {1, 2, 3, 4, 5, 6};
       int num = 4;
       int result[num];
       combine_increase(arr, 0, result, num, num, sizeof(arr)/sizeof(int));
       printf("分界线\n");
      
        NSLog(@"%@",sortalgorithm(@"Hello"));
    }
    return 0;
}
```





