# equals와 hashCode


## equals 메소드
Object 클래스에 정의된 equals 메소드는 다음과 같다. 

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```    
단순히 Object의 ==로 비교하는 것을 확인할 수 있다.

두 객체의 내용이 같은지 확인하려면 equals 메소드를 Override하면 된다.

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    AttachFile that = (AttachFile) o;
    return Objects.equals(id, that.id) && Objects.equals(fileName, that.fileName) && Objects.equals(minutes, that.minutes);
}
```

## hashCode 메소드
Object 클래스에 정의된 hashCode 메소드는 인스턴스가 다르면 구성 내용에 상관없이 전혀 다른 해시 값을 반환하도로 정의되어 있다.

```java
public static int hashCode(Object o) {
    return o != null ? o.hashCode() : 0;
}
```

```java
@HotSpotIntrinsicCandidate
public native int hashCode();
```

다음은 [hotspot의 jvm.cpp 파일에서 발췌][jvm-cpp-583]한 것이다. 버전은 JDK10 이지만, JDK8과 차이점은 없다.
```cpp
JVM_ENTRY(jint, JVM_IHashCode(JNIEnv* env, jobject handle))
  JVMWrapper("JVM_IHashCode");
  // as implemented in the classic virtual machine; return 0 if object is NULL
  return handle == NULL ? 0 : ObjectSynchronizer::FastHashCode (THREAD, JNIHandles::resolve_non_null(handle)) ;
JVM_END
```

[`JVM_IHashCode` 는 `ObjectSynchronizer::FastHashCode`][synchronizer-714]를 사용하고 있다. 그리고 이 함수를 잘 읽어보면 새로운 해시 코드 할당은 `get_next_hash` 함수를 사용한다는 것을 알 수 있다.

다음은 `get_next_hash` 함수의 코드이다.

```cpp
// hashCode() generation :
//
// Possibilities:
// * MD5Digest of {obj,stwRandom}
// * CRC32 of {obj,stwRandom} or any linear-feedback shift register function.
// * A DES- or AES-style SBox[] mechanism
// * One of the Phi-based schemes, such as:
//   2654435761 = 2^32 * Phi (golden ratio)
//   HashCodeValue = ((uintptr_t(obj) >> 3) * 2654435761) ^ GVars.stwRandom ;
// * A variation of Marsaglia's shift-xor RNG scheme.
// * (obj ^ stwRandom) is appealing, but can result
//   in undesirable regularity in the hashCode values of adjacent objects
//   (objects allocated back-to-back, in particular).  This could potentially
//   result in hashtable collisions and reduced hashtable efficiency.
//   There are simple ways to "diffuse" the middle address bits over the
//   generated hashCode values:

static inline intptr_t get_next_hash(Thread * Self, oop obj) {
  intptr_t value = 0;
  if (hashCode == 0) {
    // This form uses global Park-Miller RNG.
    // On MP system we'll have lots of RW access to a global, so the
    // mechanism induces lots of coherency traffic.
    value = os::random();
  } else if (hashCode == 1) {
    // This variation has the property of being stable (idempotent)
    // between STW operations.  This can be useful in some of the 1-0
    // synchronization schemes.
    intptr_t addrBits = cast_from_oop<intptr_t>(obj) >> 3;
    value = addrBits ^ (addrBits >> 5) ^ GVars.stwRandom;
  } else if (hashCode == 2) {
    value = 1;            // for sensitivity testing
  } else if (hashCode == 3) {
    value = ++GVars.hcSequence;
  } else if (hashCode == 4) {
    value = cast_from_oop<intptr_t>(obj);
  } else {
    // Marsaglia's xor-shift scheme with thread-specific state
    // This is probably the best overall implementation -- we'll
    // likely make this the default in future releases.
    unsigned t = Self->_hashStateX;
    t ^= (t << 11);
    Self->_hashStateX = Self->_hashStateY;
    Self->_hashStateY = Self->_hashStateZ;
    Self->_hashStateZ = Self->_hashStateW;
    unsigned v = Self->_hashStateW;
    v = (v ^ (v >> 19)) ^ (t ^ (t >> 8));
    Self->_hashStateW = v;
    value = v;
  }

  value &= markOopDesc::hash_mask;
  if (value == 0) value = 0xBAD;
  assert(value != markOopDesc::no_hash, "invariant");
  return value;
}
```

## equals와 hashCode는 왜 같이 재정의해야 할까?
hash 값을 사용하는 Collection(HashMap, HashSet, HashTable)은 객체가 논리적으로 같은지 비교할 때 아래 그림과 같은 과정을 거친다.

![그림1](https://user-images.githubusercontent.com/46465928/154830274-b1ea5d62-d21c-45db-bedd-230be24bacd6.png)

hashCode 메소드의 반환값을 이용해서 검색의 범위를 확 줄여버리고, 해당 부류 내에 존재하는 데이터의 내용 비교는 equals 메소드를 통해서 진행한다.

Object 클래스의 hashCode 메서드는 객체의 고유한 주소 값을 int 값으로 변환하기 때문에 객체마다 다른 값을 리턴한다. 두 개의 Car 객체는 equals로 비교도 하기 전에 서로 다른 hashCode 메서드의 리턴 값으로 인해 다른 객체로 판단된 것이다.

성능에 아주 민감하지 않은 대부분의 프로그램은 간편하게 Objects.hash 메서드를 사용해서 hashCode 메서드를 재정의해도 문제없다. 민감한 경우에는 직접 재정의해주는 게 좋다.

```java
@Override
    return Objects.hash(id, fileName, minutes);
}
```
```java
public static int hash(Object... values) {
    return Arrays.hashCode(values);
}
 ``` 
 
```java
public static int hashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a)
        result = 31 * result + (element == null ? 0 : element.hashCode());

    return result;
}
```

