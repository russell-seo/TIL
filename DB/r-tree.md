# R-Tree 인덱스

  R-Tree 인덱스는 `2차원 데이터`를 저장하는 인덱스이다. R-Tree 인덱스를 구성하는 컬럼의 값은 2차원 공간 개념 값으로 MySQL 공간 인덱스에 사용하게 된다.
  
  공간 인덱스는 위치 기반의 서비스를 구현할 때 주로 사용되며 MySQL의 공간 확장을 이용해 간단하게 구현할 수 있다.
  
  - MySQL의 공간 확장에는 크게 세 가지 기능이 포함 되어 있다.
  
  1. 공간 데이터를 저장할 수 있는 데이터 타입(POINT, LINE, POLYGON, GEOMETRY등)
  2. 공간 데이터 검색을 위한 공간 인덱스(R-Tree 알고리즘)
  3. 공간 데이터 연산 함수(ST_INTERSECTS, ST_CONTAINS등)



  - 간단하게 공간 데이터 타입에 대해서 알아보자.


  ![image](https://user-images.githubusercontent.com/79154652/215370686-dc4202e0-ad90-45b9-b8fd-7f4ebffdc222.png)


  - GEOMETRY 타입은 최상위 슈퍼타입 Object와 같으며 그 자식들로 우리가 사용하는 `Point`, `Polygon`등이 있다.


## MBR(Minimum Boundary Rectangle)

R-Tree 알고리즘을 이해하기 위해서는 `MBR`이라는 개념이 있는데 이전 Spatial Query Index 에서 언급한 바가 있다.
  
  ![image](https://user-images.githubusercontent.com/79154652/215373405-86283190-b9a4-403f-8f8c-08836b4c95bd.png)


- MBR은 도형을 감싸는 최소 크기의 사각형을 의미한다. 이러한 MBR의 `포함 관계를 B-Tree 형태로 구현한 인덱스가 R-Tree`이다.

- MBR이 어떻게 구성되는지 알아보면 R-Tree 인덱스 구조를 더 이해하기 쉬워질 것이다. 예를 들어 아래와 같은 공간데이터가 있다고 해보자.
 
 ![image](https://user-images.githubusercontent.com/79154652/215373595-8bd7cd1a-28b2-4f1a-a8ca-2a8cf554a425.png)

- 이제 해당 공간 데이터들을 효과적으로 검색하기 위해 인덱스를 만들어야 한다. 이때 사용되는게 `MBR`이라는 개념이다. 공간 데이터를 `MBR`로 그룹화 한 모습을 나타내는 그림이다.

![image](https://user-images.githubusercontent.com/79154652/215374011-9cf71bdf-bb9e-4712-b1dd-16b3a72ebd70.png)
  
  
    - 최상위 레벨 : R1~R2
    - 차상위 레벨 : R3~R6
    - 최하위 레벨 : R7~R14
    
  - `각 공간 데이터 들은 MBR로 둘러싸여 있고` 이 MBR 들을 그룹화한 `차상위 레벨 MBR이 존재한다` 그리고 차상위 레벨 그룹을 둘러싼 MBR이 `최상위 레벨 MBR`이 된다.

  > 이렇듯 공간을 `MBR` 그룹으로 나눔으로써 해당 공간 데이터를 찾아 들어갈 수 있는 `인덱스의 형태가 만들어진다`. 최상위 `MBR`은 루트 노드에 저장되는 정보이며, 차상위 그룹 MBR은 브랜치 노드 가 된다.
    ` 마지막으로 각 도형의 객체는 리프 노드에 저장되므로 R-Tree 인덱스 내부를 표현 할 수 있게 된다.

  ![image](https://user-images.githubusercontent.com/79154652/215374418-f61e2229-cced-4a86-bdfb-deb4003dcd1b.png)

  - 즉 앞에서 살펴 봤듯이 R-Tree는 `MBR의 포함 관계를 이용해서 만들어진 인덱스이다.` `그렇기 때문에 MySQL에서 제공하는 ST_CONTAINS, ST_WITHIN, ST_INTERSECTS등 포함 관계를 비교하는 함수로 검색 할 때만 인덱스를 사용할 수 있게 된다.`
