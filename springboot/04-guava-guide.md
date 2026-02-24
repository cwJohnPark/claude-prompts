---
name: guava-guide
description: Java Guava 라이브러리 활용 가이드. Immutable Collections, Preconditions, Multimap 등 핵심 패턴
category: springboot
tags: [spring-boot, java, guava, collections, utilities]
---

# Guava Guide

Java 코드 작성 시 Guava 라이브러리를 활용하여 더 간결하고 안전한 코드를 작성합니다.

**Version**: 33.5.0-jre
**Documentation**: https://github.com/google/guava/wiki

## 활성화 조건

Java 코드를 작성할 때 항상 이 스킬을 참고합니다.

## 핵심 원칙

1. **Null Safety**: Optional, Preconditions 활용
2. **Immutability**: Immutable Collections 우선 사용
3. **Functional Style**: 함수형 프로그래밍 패턴 지원
4. **Type Safety**: 강타입 유틸리티 활용

---

## 1. Basic Utilities

### 1.1 Using/Avoiding Null (Optional)

```java
import com.google.common.base.Optional;

// Guava Optional 사용 (Java Optional과 유사)
Optional<String> optional = Optional.of("value");
Optional<String> absent = Optional.absent();

// null 처리
String value = Optional.fromNullable(nullableString).or("default");
```

### 1.2 Preconditions

```java
import static com.google.common.base.Preconditions.*;

public void process(List<String> items, int index) {
    checkNotNull(items, "items must not be null");
    checkArgument(!items.isEmpty(), "items must not be empty");
    checkElementIndex(index, items.size(), "index");
    checkState(isInitialized(), "must be initialized first");
}
```

### 1.3 Object Methods

```java
import com.google.common.base.Objects;
import com.google.common.base.MoreObjects;
import com.google.common.collect.ComparisonChain;

// equals helper
Objects.equal(a, b);  // null-safe equals

// toString helper
@Override
public String toString() {
    return MoreObjects.toStringHelper(this)
            .add("id", id)
            .add("name", name)
            .toString();
}

// compareTo helper
@Override
public int compareTo(Person other) {
    return ComparisonChain.start()
            .compare(lastName, other.lastName)
            .compare(firstName, other.firstName)
            .compare(age, other.age)
            .result();
}
```

### 1.4 Ordering

```java
import com.google.common.collect.Ordering;

// 정렬 유틸리티
Ordering<String> byLength = Ordering.natural()
        .onResultOf(String::length)
        .nullsFirst();

List<String> sorted = byLength.sortedCopy(strings);
String max = byLength.max(strings);
String min = byLength.min(strings);
```

---

## 2. Collections

### 2.1 Immutable Collections (권장)

```java
import com.google.common.collect.ImmutableList;
import com.google.common.collect.ImmutableSet;
import com.google.common.collect.ImmutableMap;

// 생성
ImmutableList<String> list = ImmutableList.of("a", "b", "c");
ImmutableSet<String> set = ImmutableSet.of("a", "b", "c");
ImmutableMap<String, Integer> map = ImmutableMap.of("a", 1, "b", 2);

// Builder 패턴
ImmutableList<String> builtList = ImmutableList.<String>builder()
        .add("a")
        .addAll(existingList)
        .build();

// copyOf
ImmutableList<String> copied = ImmutableList.copyOf(mutableList);
```

### 2.2 New Collection Types

#### Multiset (원소 개수 추적)
```java
import com.google.common.collect.HashMultiset;
import com.google.common.collect.Multiset;

Multiset<String> multiset = HashMultiset.create();
multiset.add("apple", 3);  // 3개 추가
int count = multiset.count("apple");  // 3
```

#### Multimap (키당 여러 값)
```java
import com.google.common.collect.ArrayListMultimap;
import com.google.common.collect.Multimap;
import com.google.common.collect.ImmutableListMultimap;

Multimap<String, String> multimap = ArrayListMultimap.create();
multimap.put("fruit", "apple");
multimap.put("fruit", "banana");
Collection<String> fruits = multimap.get("fruit");  // [apple, banana]

// Immutable 버전
ImmutableListMultimap<String, String> immutableMultimap =
    ImmutableListMultimap.of("fruit", "apple", "fruit", "banana");
```

#### BiMap (양방향 맵)
```java
import com.google.common.collect.HashBiMap;
import com.google.common.collect.BiMap;

BiMap<String, Integer> biMap = HashBiMap.create();
biMap.put("one", 1);
String key = biMap.inverse().get(1);  // "one"
```

#### Table (2차원 맵)
```java
import com.google.common.collect.HashBasedTable;
import com.google.common.collect.Table;
import com.google.common.collect.ImmutableTable;

Table<String, String, Integer> table = HashBasedTable.create();
table.put("row1", "col1", 1);
table.put("row1", "col2", 2);

Integer value = table.get("row1", "col1");  // 1
Map<String, Integer> row = table.row("row1");  // {col1=1, col2=2}
Map<String, Integer> col = table.column("col1");  // {row1=1}
```

#### RangeSet & RangeMap
```java
import com.google.common.collect.Range;
import com.google.common.collect.TreeRangeSet;
import com.google.common.collect.TreeRangeMap;

// RangeSet
TreeRangeSet<Integer> rangeSet = TreeRangeSet.create();
rangeSet.add(Range.closed(1, 10));
rangeSet.add(Range.closed(15, 20));
boolean contains = rangeSet.contains(5);  // true

// RangeMap
TreeRangeMap<Integer, String> rangeMap = TreeRangeMap.create();
rangeMap.put(Range.closed(1, 10), "low");
rangeMap.put(Range.closed(11, 20), "high");
String grade = rangeMap.get(5);  // "low"
```

### 2.3 Collection Utilities

```java
import com.google.common.collect.Iterables;
import com.google.common.collect.Lists;
import com.google.common.collect.Sets;
import com.google.common.collect.Maps;

// Iterables
Iterables.getLast(list);
Iterables.getOnlyElement(singleItemList);
Iterables.partition(list, 10);  // 10개씩 분할

// Lists
List<List<String>> partitions = Lists.partition(list, 10);
List<String> reversed = Lists.reverse(list);

// Sets
Set<String> union = Sets.union(set1, set2);
Set<String> intersection = Sets.intersection(set1, set2);
Set<String> difference = Sets.difference(set1, set2);

// Maps
ImmutableMap<String, Person> byName = Maps.uniqueIndex(people, Person::getName);
```

---

## 3. Strings

### 3.1 Joiner & Splitter

```java
import com.google.common.base.Joiner;
import com.google.common.base.Splitter;

// Joiner
String joined = Joiner.on(", ")
        .skipNulls()
        .join(Arrays.asList("a", null, "b"));  // "a, b"

// Map 조인
String mapString = Joiner.on("; ")
        .withKeyValueSeparator("=")
        .join(map);  // "key1=value1; key2=value2"

// Splitter
List<String> parts = Splitter.on(",")
        .trimResults()
        .omitEmptyStrings()
        .splitToList("a, b,,  c");  // ["a", "b", "c"]

// Map 분할
Map<String, String> parsedMap = Splitter.on(";")
        .withKeyValueSeparator("=")
        .split("key1=value1;key2=value2");
```

### 3.2 CharMatcher

```java
import com.google.common.base.CharMatcher;

// 문자 필터링
String digitsOnly = CharMatcher.digit().retainFrom("abc123def");  // "123"
String noDigits = CharMatcher.digit().removeFrom("abc123def");  // "abcdef"
String collapsed = CharMatcher.whitespace().collapseFrom("a  b   c", ' ');  // "a b c"
String trimmed = CharMatcher.whitespace().trimFrom("  hello  ");  // "hello"

// 검증
boolean hasDigit = CharMatcher.digit().matchesAnyOf("abc123");  // true
```

### 3.3 CaseFormat

```java
import com.google.common.base.CaseFormat;

// 케이스 변환
String camel = CaseFormat.LOWER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "my_variable");  // "myVariable"
String snake = CaseFormat.LOWER_CAMEL.to(CaseFormat.LOWER_UNDERSCORE, "myVariable");  // "my_variable"
String constant = CaseFormat.LOWER_CAMEL.to(CaseFormat.UPPER_UNDERSCORE, "myVariable");  // "MY_VARIABLE"
```

---

## 4. Caching

```java
import com.google.common.cache.CacheBuilder;
import com.google.common.cache.CacheLoader;
import com.google.common.cache.LoadingCache;

// 캐시 생성
LoadingCache<Key, Value> cache = CacheBuilder.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .expireAfterAccess(5, TimeUnit.MINUTES)
        .recordStats()
        .build(new CacheLoader<Key, Value>() {
            @Override
            public Value load(Key key) {
                return expensiveComputation(key);
            }
        });

// 사용
Value value = cache.get(key);  // 캐시에 없으면 load() 호출
cache.invalidate(key);  // 특정 키 무효화
cache.invalidateAll();  // 전체 무효화
```

---

## 5. Concurrency

### 5.1 ListenableFuture

```java
import com.google.common.util.concurrent.ListenableFuture;
import com.google.common.util.concurrent.ListeningExecutorService;
import com.google.common.util.concurrent.MoreExecutors;
import com.google.common.util.concurrent.Futures;

// ListeningExecutorService 생성
ListeningExecutorService service = MoreExecutors.listeningDecorator(
        Executors.newFixedThreadPool(10));

// Future 생성
ListenableFuture<Result> future = service.submit(() -> compute());

// 콜백 추가
Futures.addCallback(future, new FutureCallback<Result>() {
    @Override
    public void onSuccess(Result result) {
        handleSuccess(result);
    }

    @Override
    public void onFailure(Throwable t) {
        handleFailure(t);
    }
}, executor);

// 변환
ListenableFuture<Output> transformed = Futures.transform(
        future, input -> convert(input), executor);

// 여러 Future 결합
ListenableFuture<List<Result>> allResults = Futures.allAsList(futures);
```

---

## 6. Primitives & Math

```java
import com.google.common.primitives.Ints;
import com.google.common.primitives.Doubles;
import com.google.common.math.IntMath;
import com.google.common.math.DoubleMath;

// Primitives
int[] array = Ints.toArray(integerList);
List<Integer> list = Ints.asList(1, 2, 3);
int max = Ints.max(array);
int min = Ints.min(array);
boolean contains = Ints.contains(array, 5);

// Math (overflow-safe)
int sum = IntMath.checkedAdd(a, b);  // ArithmeticException on overflow
int pow = IntMath.pow(2, 10);  // 1024
int log2 = IntMath.log2(16, RoundingMode.FLOOR);  // 4
boolean isPowerOf2 = IntMath.isPowerOfTwo(8);  // true
```

---

## 7. I/O & Hashing

### 7.1 Files & Resources

```java
import com.google.common.io.Files;
import com.google.common.io.Resources;

// 파일 읽기/쓰기 (deprecated - Java NIO 권장)
// Files.asCharSource(), Files.asCharSink() 사용

// 리소스 읽기
URL url = Resources.getResource("config.properties");
String content = Resources.toString(url, StandardCharsets.UTF_8);
```

### 7.2 Hashing

```java
import com.google.common.hash.Hashing;
import com.google.common.hash.HashCode;
import com.google.common.hash.BloomFilter;
import com.google.common.hash.Funnels;

// 해싱
HashCode hash = Hashing.sha256().hashString("input", StandardCharsets.UTF_8);
String hexHash = hash.toString();

// BloomFilter
BloomFilter<String> filter = BloomFilter.create(
        Funnels.stringFunnel(StandardCharsets.UTF_8), 1000, 0.01);
filter.put("element");
boolean mightContain = filter.mightContain("element");  // true
```

---

## 8. EventBus

```java
import com.google.common.eventbus.EventBus;
import com.google.common.eventbus.Subscribe;

// EventBus 생성
EventBus eventBus = new EventBus();

// 구독자 등록
public class EventHandler {
    @Subscribe
    public void handleEvent(MyEvent event) {
        // 이벤트 처리
    }
}

eventBus.register(new EventHandler());

// 이벤트 발행
eventBus.post(new MyEvent());
```

---

## 9. Ranges

```java
import com.google.common.collect.Range;
import com.google.common.collect.BoundType;

// Range 생성
Range<Integer> closed = Range.closed(1, 10);      // [1, 10]
Range<Integer> open = Range.open(1, 10);          // (1, 10)
Range<Integer> closedOpen = Range.closedOpen(1, 10);  // [1, 10)
Range<Integer> atLeast = Range.atLeast(5);        // [5, +∞)
Range<Integer> lessThan = Range.lessThan(10);     // (-∞, 10)

// Range 연산
boolean contains = closed.contains(5);  // true
boolean encloses = closed.encloses(Range.closed(2, 5));  // true
Range<Integer> intersection = closed.intersection(Range.closed(5, 15));  // [5, 10]
Range<Integer> span = closed.span(Range.closed(15, 20));  // [1, 20]
```

---

## 프로젝트 적용 가이드

### 권장 사용 패턴

```java
// 1. Preconditions로 입력 검증
public Result calculate(Distance distance, Consumption consumption) {
    checkNotNull(distance, "distance must not be null");
    checkNotNull(consumption, "consumption must not be null");
    checkArgument(distance.isPositive(), "distance must be positive");
    // ...
}

// 2. Immutable Collections 사용
public ImmutableList<DailyRecord> getReports() {
    return ImmutableList.copyOf(reports);
}

// 3. Multimap으로 그룹화
Multimap<UserId, Report> reportsByUser =
    ArrayListMultimap.create();

// 4. Range로 기간/범위 표현
Range<LocalDate> period = Range.closed(startDate, endDate);
boolean inPeriod = period.contains(targetDate);

// 5. Table로 2차원 데이터 관리
Table<UserId, YearMonth, Rating> ratings = HashBasedTable.create();
```

### 주의사항

1. **Java 표준 라이브러리와 중복되는 기능**은 표준 라이브러리 우선 사용
   - `java.util.Optional` vs `com.google.common.base.Optional`
   - `java.util.stream.Collectors` vs Guava collectors

2. **Immutable Collections**은 null 값을 허용하지 않음

3. **Deprecated 기능 사용 자제**
   - `Files` 클래스의 일부 메서드 -> Java NIO 사용
