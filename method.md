## Method

Go에서 `객체`는 메소드를 가진 값이나 변수를, `메소드`는 특정 타입과 관련된 함수를 의미한다. 

### Receiver

* Method는 함수 + Receiver의 구조로 되어있다. 함수명 앞에 해당 메소드를 사용할 **타입** 이름을 명시해준다.

  Go는 어떤 타입(숫자, 문자열, 슬라이스, Map, 함수)에도 메소드를 붙일 수 있다

* Receiver의 이름은 짧게 (보통 타입의 첫글자를 붙임)

* 한 타입안에서 메소드명은 유일해야 함 

```go
type Rect struct {
	width, height int
}

// Rect 타입의 메소드
func (r Rect) Area() int {
	return r.width * r.height
}

// 동일한 역할을 수행하는 그냥 function
func RectArea(r Rect) int {
	return r.width * r.height
}

func main() {
	r := Rect{width: 2, height: 3}

	fmt.Println("Area is ", r.Area())
	fmt.Println("Area is ", RectArea(r))
	// 메소드를 쓰면 함수 이름이 더 짧아질 수 있다는 장점이 있음
}
```



#### Value / Pointer Receiver

```go
// Value
// 메소드를 호출시, 호출한 애의 값들이 복사되고 그 값을 바탕으로 수행
// 메소드 안에서 값을 바꿔도, 호출한 애한테 영향 X
func (r Rect) ChangeWidth(w int){
  r.width = w
}

// Pointer
// 호출한 애의 주소값이 복사됨 
func (r *Rect) ChangeWidthPtr(w int){
  (*r).width = w
}

func main() {
	r := Rect{width: 2, height: 3}
	r.ChangeWidth(10)
	fmt.Println("Width is ", r.width)	// 2

	(&r).ChangeWidthPtr(10)
	fmt.Println("Width is ", r.width)	// 10
  
  // 포인터 타입으로 호출하지 않아도 암묵적으로 포인터 타입으로 바꿔준다
  r.ChangeWidthPtr(20)	  
  fmt.Println("Width is ", r.width)	// 20
}

```

* Value receiver는 함수 호출마다 매번 값이 복사되어 사용된다 ( 메모리 낭비가 있을 수 있다 ) 

  `인스턴스 복사가 안전하다`

* Pointer receiver는 주소 값이 복사되어 가고, 이 주소값을 통해 해당 객체에 접근하여 연산을 수행한다. 

  `포인터 타입으로 호출하지 않아도  go에서 자동으로 타입을 바꿔준다 `

* 한 타입 안에 Pointer Receiver의 메소드가 있다면, 다른 모든 메소드들도 Pointer Receiver로 해주는 것이 관행이다!

  

  #### nil

  * nil이 유의미한 제로값으로 Receiver에 들어올 수 있다. 

  ```go
  func (list *List) Sum() int{
    // 빈 리스트인 경우
    if list == nil{
    	return 0
    }
    
    return list.Value + list.Tail.Sum()
  }
  ```



### Embedding Structure

```go
type ColoredRect struct{
	Rect
	color int
}

func (s ColoredRect) print(){
	fmt.Println("print")
}


func main() {
	r := Rect{width: 2, height: 3}
	cs := ColoredRect{Rect: r, color:1}

	cs.ChangeWidthPtr(10)
	cs.Area()
	cs.print()
	
	// RectArea(cs) Error! 
}
```

* Structure Embedding이 자바의 상속과 완전 같은 개념이라고는 볼 수 없다

  * `ColoredRect가 Rect 종류 중 하나` 라는 의미가 아니라 `ColoredRect가 Rect를 가지고 있다`는 의미

    "ColoredRect는 Rect는 아니지만, Rect를 가지고 있고 && Rect의 메소드를 갖고 있다"

* ColoredRect가 Rect의 메소드를 **바로** 사용할 수 있도록, 컴파일러가 내부적으로 아래와 같은 Wrapper 메소드를 생성해준다고 보면된다. 

```go
func (cs ColoredRect) Area() int{
  return cs.Rect.Area()
}

func (cs *ColoredRect) ChangeWidthPtr(w int){
  cs.Rect.ChangeWidthPtr(w)
}
```



### 캡슐화

객체의 사용자가 객체의 변수나 메소드에 접근할 수 없는 경우 객체가 **캡슐화** 되어 있다고 말한다. 

* Go에서는 이름의 대소문자로 접근 제한을 제어한다

  ```go
  // 어디서나 사용 가능
  func (r *Rect) ChangeWidthPtr(w int){
    (*r).width = w
  }
  
  // Rect 정의한 외부 패키지에서는 사용 불가
  func (r *Rect) changeHeight( h int){
    (*r).height = h
  }
  ```