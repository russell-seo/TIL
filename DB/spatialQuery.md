# Spatial Query(공간 정보 쿼리)

  - Mysql 에서 좌표(Point), 다각형(Polygon)등 같은 공간 기하 데이터를 다루는 쿼리를 지원한다.
  - 필자는 공간 관련된 프로토타입을 만들어야해서 어플리케이션에서 처리하는게 아닌 DB의 공간 관련된 쿼리를 사용해보고 기술해볼려고 한다.


## 공간 데이터 타입

  - 먼저 Mysql 에서 지원하는 공간 데이터 타입에 대해 살펴보자
    - Point : 좌표 공간에서 한 지점의 위치를 표시하는 타입 -> `POINT(10 10)`
    - LineString : 다수의 Point 를 연결해주는 선분 -> `LINESTRING(10 10, 20 25, 15 40)`
    - Polygon : 다수의 선분들이 연결되어 닫혀 있는 상태인 다각형 -> `POLYGON((10 10, 10 20, 20 20, 20 10, 10 10))`
    - Multi-Point : 다수 개의 Point 집합 -> `MULTIPOINT (10 10, 30 20)`
    - Multi-LineString : 다수 개의 LineString 집합 -> `MULTILINESTRING((10 10, 20 20), (20 15, 30 40))`
    - Multi-Polygon : 다수 개의 Polygon 집합 -> `MULTIPOLYGON((10 10, 10 20, 20 20, 20 10, 10 10), (10 10, 10 20, 20 20, 20 10, 10 10))`
    - GeomCollection : 모든 공간 데이터들의 집합 -> `GEOMETRYCOLLECTION(POINT(10 10), LINESTRING(20 20, 30 40), POINT(30 15))`
    
    
    ![image](https://user-images.githubusercontent.com/79154652/214288762-1ec1f11a-f3e3-43c8-8d2d-6a874ec905ba.png)


## 공간 관계 함수

  - 공간 데이터에 대해 알아봤으며 공간 데이터를 DB에서 CRUD 할수 있는 함수에 대해 알아보자.
    - ST_Equals(g1 GeoMetry, g2 GeoMetry) : Boolean -> g1과 g2가 동일하면 True를 반환하고 상이하면 False
    - ST_Disjoint(g1, g2) : Boolean -> g1과 g2가 겹치는 곳이 없다면 True 아니면 False
    - ST_Within(g1, g2) : Boolean -> g1이 g2 영역안에 포함되어있으면 True OrNot False
    - ST_Overlaps(g1, g2) : Boolean -> g1과 g2 영역 간에 교집합이 존재하면 True OrNot False
    - ST_Intersects(g1, g2) : Boolean -> g1과 g2가 교집합이 존재하는 경우 True OrNot False
    - ST_Contains(g1, g2) : Boolean -> g2가 g1 영역안에 포함된 경우 Treu OrNot False
    - ST_Touches(g1, g2) : Boolean -> g1과 g2 경계 영역에서만 겹치는 경우 결과 값으로 True orNot False
    - ST_Distance(g1, g2) : Double -> g1과 g2간의 거리 반환
    - ST_Distance_Sphere(g1, g2) : Double -> g1, g2 거리 반환



## 공간 데이터, 공간 함수 사용하여 DB TEST

  - 공간 데이터와 공간 함수에 대해 알아봤으므로 직접 DB에 테이블을 만들고 사용해보는 실습을 해보겠다.

  1. 먼저 테이블 생성
  ~~~mysql
 CREATE TABLE geo (
  `id` bigint not null auto_increment,
  `name` varchar(200) not null,
  `coordinates` point not null,
  primary key(`id`)
 )
  ~~~
  
  - 위에서 주의깊게 봐야할 것이 바로 `Point` 라는 데이터 타입이다.

  2. 해당 테이블에 데이터 Insert
  ~~~mysql
  insert into geo (name, coordinates) values
  (`Eiffel Tower`, POINT(48.858271, 2.293795));
  ~~~
  
  - 에펠탑에 대한 위/경도 좌표를 넣으면 아래와 같이 데이터가 들어가게 된다.
![스크린샷 2023-01-24 오후 9 43 49](https://user-images.githubusercontent.com/79154652/214294858-7c43fd52-b2ae-4e77-b14a-9915522eb85b.png)


  3. 위의 데이터를 가지고 위/경도 좌표간의 거리를 구하는 함수를 사용해보자.
    - 필자는 ST_Distance_Sphere 함수를 사용해서 거리를 구할 것이다.
  ~~~
  select name, ST_Distance_Sphere(coordinates, Point(48.861105, 2.335337)) from geo
  ~~~
  
   - Point(48.861105, 2.335337)은 루브르 박물관의 위/경도 좌표이며 에펠탑과의 거리를 구하는 쿼리이다.
   ![스크린샷 2023-01-24 오후 9 52 17](https://user-images.githubusercontent.com/79154652/214296552-8ea458af-0d04-4292-9a93-6fc3d53c2def.png)

  
  
 ### Polygon
 
  - Polygon 타입의 좌표로 이루어진 다각형 안에 해당 좌표가 위치하는지에 대해 테스트 해 볼려고 한다.


  1. Polygon 타입의 테이블을 생성
  ~~~
  create table polygon (
    `id` bigint not null auto_increment,
    `name` varchar not null,
    `polygon` polygon not null
    primary key(`id`)
  )
  ~~~
  
  
  2. Polygon 데이터 Insert
  ~~~mysql
  insert into polygon (polygon, name)
  values(ST_GEOMFROMTEXT('POLYGON((48.89 2.27, 48.89 2.42, 48.81 2.42, 48.81 2.27, 48.89 2.27))', `PARIS`);
  ~~~
  - `ST_GeomFromText` 함수는 잘 알려진 텍스트(WKT) 포인트 표시에서 포인트, 라인스트링 및 폴리곤을 작성하고 삽입하는 데 사용됩니다.
  - 해당 Polygon 좌표들은 실제 파리의 좌표를 가져온 것 입니다.


  3. ST_Contains 함수 활용
   - 해당 함수를 활용하여 좌표가 해당 Polygon 안에 위치하는지 확인할 수 있다.
    
   ~~~mysql
    select name from polygon a where st_contains(a.polygon, Point(48.861105, 2.335337));
   ~~~
    
   - Point(48.861105, 2.335337) 는 루브르 박물관의 위/경도 좌표이다.
   - 해당 쿼리는 아래와 같이 `PARIS`라는 로우가 조회가된다.

   ![스크린샷 2023-01-24 오후 10 07 36](https://user-images.githubusercontent.com/79154652/214300176-a8aff574-d980-479e-8cb6-75face3dcd94.png)
   
   - 만약 다른 나라의 좌표를 넣는다고 하면 어떻게 조회되는지 확인해 보자.
   
   ![스크린샷 2023-01-24 오후 10 11 45](https://user-images.githubusercontent.com/79154652/214302830-1148226a-38c5-4476-a29c-a6acafebe7c1.png)
  
  - 구글에서 프랑스의 파리가 아닌 리옹의 아무곳이나 찍어 위/경도를 가져와서 쿼리를 돌려봤다.
  
  ~~~mysql
  select name from polygon a where st_contains(a.polygon, Point(45.928543, 4.605636));
  ~~~
![image](https://user-images.githubusercontent.com/79154652/214303713-5d9eede3-d83a-49fe-b3df-78428c71f337.png)


  - 결과는 Null 값을 리턴한다. 즉 해당 좌표는 해당 Polygon에 속하지 않는다는 것이다.


## 마무리

- 일단 오늘은 공간데이터 타입과 함수를 알아 보았고 필자는 회사에서 계속 이와 관련된 프로젝트를 진행하며 공부해 나가야 하기때문에 이에 관한 정보는 계속 기술해 나갈 예정이다.
