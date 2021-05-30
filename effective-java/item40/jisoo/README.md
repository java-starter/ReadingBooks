# [ITEM 40] @Override 애너테이션을 일관 되게 사용하라.
#### @Override ?
자바가 기본으로 제공하는 애너테이션중 가장중요한것.
메서드 선언에만 달 수 있으며, 이 애너테이션이 달렷다는 것은 상위 타입의 메서드를 재저으이했음을 뜻한다. 

```java
public class Bigram {  
    private final char fisrt;  
 private final char second;  
  
 public Bigram(char first , char second){  
        this.fisrt = first;  
 this.second = second;  
 }  
  
    public boolean eqauls(Bigram b){  
        return b.fisrt == fisrt && b.second == second;  
 }  
  
    public int hashCode(){  
        return 31 * fisrt + second;  
 }  
  
    public static void main(String\[\] args){  
        Set<Bigram> s = new HashSet<>();  
 for(int i = 0; i < 10 ; i++){  
            for (char ch = 'a'; ch <= 'z' ; ch++){  
                s.add(new Bigram(ch,ch));  
 }  
            System.out.println(s.size());  
 }  
    }  
}
```


set 은 중복을 허용하지 않으므로  26이 출려되어야 하지만 260이 출력됨
--> eqaul메서드를 overrding 한게 아니라 overloading 하였다.  eqauls를 재정의 하려면 매개 변수 타입을 Object로 해야만한다. 
--> 하지만 @Override 어노테이션을 달면 컴파일 오류가 발생하여 곧장 올바르게 수정할 수 있다. 

```java
@Override  
public boolean equals(Object o){  
    if(!(o instanceof Bigram)){  
        return false;  
 }  
    Bigram b = (Bigram) o;  
 return b.fisrt \== fisrt && b.second \== second;  
}
```

상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자