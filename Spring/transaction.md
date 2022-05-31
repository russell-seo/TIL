
# Spring Transaction VS Spring Boot Transaction


- `@Transactional`은 많은 편리성을 제공해준다.
    - transcation `begin`, `commit`을 자동 수행해준다.
    - 예외를 발생시키면, `rollback`처리를 자동 수행해준다.
  
  ~~~java
  public class BooksImpl implements Books {  
  @Transactional  
  public void addBook(String bookName) {
  Book book = new Book(bookName);    
  bookRepository.save(book);    
  book.setFlag(true);  
   }
  }
  ~~~
  
  ~~~java
  public class BooksImpl implements Books {
  public void addBooks(List<String> bookNames) {
  bookNames.forEach(bookName -> this.addBook(bookName));  
  }
  
  @Transactional  
  public void addBook(String bookName) {    
  Book book = new Book(bookName);    
  bookRepository.save(book);    
  book.setFlag(true);  
  }}
  ~~~
  
  위의 코드에서 발생하는 문제는 `addBooks` 메소드 내부에서 호출하는 `addBook`메소드의 `@Transactional` 어노테이션이 적용되지 않는 것이다.
  
  그렇기에, 해당 코드를 실행하더라도 DB에 저장된 Book 정보의 Flag 컬럼에 업데이트 되지 않는다.
  
  bookRepository의 save 메소드는 자체적으로 `@Transactional` 어노테이션이 붙어 있어 정상적으로 코드가 수행되어 DB에 Insert 한다.
  
  하지만 `update` 역할을 하는 Book 객체의 변경감지가 동작하지 않아 Flag 컬럼에 업데이트가 되지 않는다.
  
  
  ## Spring @Transactional 기능 제공 방식
  
  JPA의 객체 변경감지는 transaction 이 commit 될 때 작동합니다.
  
  그렇기에 Spring은 `@Transactional` 어노테이션을 선언한 메소드가 실행되기전, transaction begin 코드를 삽입하여, 메소드가 실행된 후,
  `Transaction commit` 코드를 삽입하여, 객체 변경감지를 수행하게 유도한다.
  
  `Spring`의 코드 삽입 방법은 크게 2가지 방법이 있다.
  
  - 바이트 코드 생성(CGLIB 사용)
  - 프록시 객체 사용
  
  > 2가지 방법중 `spring`은 기본적으로 `프록시 객체 사용` 이 선택된다. 그렇기에 `interface`가 반드시 필요하다.
  > `SpringBoot`는 기본적으로 `바이트코드 생성`이 선택됩니다. 그렇기에 굳이 `Interface`가 필요없습니다. 만약 개발환경이 `SpringBoot`라면 `Books interface` 없이 봐도 무방합니다.
  
  
  원리는 아래와 같다. 프록시 객체로 우리가 만든 메서드를 한번 감싸서, 메서드 위 아래로 코드를 삽입해 줍니다.
  ~~~java
  public class BooksProxy {  
  private final Books books;  
  private final TransactonManager manager = TransactionManager.getInstance();    
  
  public BooksProxy(Books books) {    
  this.books = books;  
  }    
  
  public void addBook(String bookName) {    
  try {      
  manager.begin();      
  books.addBook(bookName);      
  manager.commit();    } 
  catch (Exception e) {
  manager.rollback();    
  }  
  }
  }
 
  ~~~

  우리는 `BookImpl`자료형을 사용할 때, 스프링이 제공하는 BooksProxy 객체를 사용하게 되며, BooksProxy 객체가 제공하는 addBook 메소드를 사용해야만 transaction 처리가 수행되는것
  
  ## @Transactional 이 수행되지 않은 이유
  
  ~~~java
  public class BooksImpl implements Books {  
    public void addBooks(List<String> bookNames) {    
    bookNames.forEach(bookName -> this.addBook(bookName));  
    }    
    
    @Transactional  public void addBook(String bookName) {
      Book book = new Book(bookName);    
      bookRepository.save(book);    
      book.setFlag(true);  
        }
       }
  ~~~
  
  `BooksProxy`가 `addBooks` 메소드를 수행하면, 아래와 같은 순서로 작동됩니다.
  
  `BooksProxy::addBooks` -> `BooksImpl::Book`
  
  즉, `BooksImpl` 내부의 코드(addBook) 가 수행되기 때문에 해당 메서드는 프록시로 감싸진 메서드가 아니라는 점에서 `Transactional` 어노테이션 기능이 수행되지 않는 것이다.
  
  ## 해결방법
  
  사실 `@Transactional` 메서드를 내부적으로 사용하지 않는 것이 근본적인 해결책이다.
  
  굳이 사용해야 겠다면, 의존성 주입을 이용하여 `Proxy 인스턴스`를 자체적으로 가져와 사용하는 방법이 있다.
  
  ~~~java
  @Service
  public class BooksImpl implements Books{
    @Autowired
    private Books self;
    
    public void addBooks(List<String> bookNames) {
        bookNames.foreach(bookName -> self.addBook(bookName)); // this가 아닌 변수 self로
      }
      
    
    @Transactional
    public void addBook(String bookName){
        Book book = new Book(bookName);
        bookRepository.save(book);
        book.setFlag(true);
    
      }
  }
  ~~~
  
  해당 코드는 Books 인터페이스를 이용하여 BooksProxy 인스턴스를 주입할 수 있도록 유도한다.(생성자 주입을 사용하면 순환에러가 발생)
  
  그 후, 즉 자기자신 `this`가 가지고 있는 순수한 addBook 메서드가 아니라 proxy로 감싸진 addBook 메서드를 통해 `@Transactional` 어노테이션 기능을 사용할 수 있게 된다.
  
  ## 정리하자면
  
 - `@Transactional` 어노테이션 메서드는 클래스 내부적으로 사용하지말고, 밖에서 사용하자.(프록시 객체 때문)
 - 굳이 내부적으로 사용하려면, 자기자신 Proxy 객체를 사용하여 처리하자.
