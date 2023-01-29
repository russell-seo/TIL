# Spatial Query Index 최적화

  - 이전 글에서 보면 Spatial Query와 Mysql 에서 제공하는 함수를 이용하여 간단한 쿼리에 대한 이해와 실습을 하였다.
  - 이번 글에서는 Spatail Query 인덱스를 태워서 성능 개선을 해보겠다.

  먼저 실습하기 전에 SRID 라는 것에 대해 미리 알고 가보자

  ## SRID(Spatial Reference Identifier)
  
  공간 인스턴스에는 SRID(spatial reference identifier)가 있습니다. SRID는 평면 지구 매핑 또는 둥근 지구 매핑에 사용되는 특정 타원면을 기준으로 하는 공간 참조 시스템에 해당합니다.

  공간 열은 SRID가 다른 개체를 포함할 수 있습니다. 그러나 데이터에 대한 SQL Server 공간 데이터 메서드를 사용하여 작업을 수행할 때는 SRID가 동일한 공간 인스턴스만 사용할 수 있습니다. 

  `두 공간 데이터 인스턴스에서 파생되는 모든 공간 메서드의 결과는 이러한 인스턴스에 동일한 SRID가 있을 경우에만 유효합니다.` 이 SRID는 인스턴스의 좌표를 결정하는 데 사용되는 동일한 측정 단위, 데이터 및 프로젝션을 기준으로 합니다. 가장 일반적인 SRID의 측정 단위는 미터 또는 제곱미터입니다.

  두 공간 인스턴스의 SRID가 같지 않을 경우 이 인스턴스에서 사용되는 geometry 또는 geography 데이터 형식 메서드의 결과로 NULL이 반환됩니다. 예를 들어 NULL이 아닌 결과를 반환하는 다음 조건자 조건의 경우 및 의 두 geometry1 geometry geometry2인스턴스의 SRID가 같아야 합니다.
  
![image](https://user-images.githubusercontent.com/79154652/215309361-6fab0de3-d4f5-47ff-999c-ee18856f061c.png)

> 공식문서 상에 나온 글을 해석해 보면 `비교가 제대로 작동하려면 SPATIAL인덱스의 각 열이 SRID로 제한되어야 합니다. 즉, 열 정의는 명시적 SRID속성을 포함해야 하며 모든 열 값은 동일한 SRID를 가져야 합니다.`

![image](https://user-images.githubusercontent.com/79154652/215309495-f3e73d81-ef32-4113-a763-6022146efe55.png)

> 즉 테이블을 생성할 때 SRID 값을 지정해서 테이블을 생성해주어야 하며 SRID 값이 존재하지 않는 로우는 인덱는 무시해버린다.
  
  아래 Mysql 공식문서에서 해당 SRID에 관련된 내용을 참고할 수 있다.
  
  [https://dev.mysql.com/doc/refman/8.0/en/spatial-index-optimization.html](https://dev.mysql.com/doc/refman/8.0/en/spatial-index-optimization.html)
  
  `GPS 의 기준이 되는 WGS84 시스템은 SRID 4326, 단순 직교 좌표계는 SRID 0 입니다.`
  
  
  ## Spatial Query Table 생성 및 쿼리 실행
  
  - 먼저 GeoJson 데이터를 저장할 테이블을 생성한다.
  - 필자는 country 라는 테이블을 생성하였다.
  
  ![image](https://user-images.githubusercontent.com/79154652/215325948-95c1d1ce-33f0-49f5-b1ae-14da0d6fbc1c.png)

  - `code`는 각 국가의 국가번호를 의미하며, `polygon`은 polygon 타입의 공간데이터 타입으로 생성하였다.
  - 추가로 생성한 테이블에 GeoJson을 다운받아 파싱하여 데이터를 저장한 상태이다.
  
  
  1. 먼저 필자는 현재 나의 위치와 폴리곤 혹은 두 좌표를 받아 MBR을 생성하고 교차하는 폴리곤을 모두 클라이언트에 내려줘야하는 상황이다.
  2. 교차하는 폴리곤을 찾기위해 Mysql 에서 제공하는 공간 함수는 `ST_INTERSECTS` 이다. `ST_INTERSECTS`는 폴리곤과 교차하는 점이 있으면 TRUE를 없으면 False를 반환하는 함수이다` 자세한 내용은 아래 링크를 참고하면 될 것 이다.
     [공간 함수](https://dev.mysql.com/doc/refman/5.7/en/spatial-function-reference.html)
  3. 추가로 해당 테이블에 `polygon` 컬럼에 인덱스를 추가하였다. `참고로 Mysql 에서도 공간데이터에 대한 인덱스를 지원한다`
    ![image](https://user-images.githubusercontent.com/79154652/215327405-2feb1f1b-020f-40d3-b4e3-959ff2d4bb70.png)

  4. 이제 DB에서 실습을 해보자. 아래 쿼리는 하나의 폴리곤과 DB에 있는 polygon 컬럼이 교차하는 도시와 나라의 이름을 조회하는 쿼리이다.
  5. 아래 파라미터로 넣은 폴리곤은 지도상에서 내가 직접 테스트하기 위해 그린 MBR(Minimum Boundary Rectangle) 이다.
     ![image](https://user-images.githubusercontent.com/79154652/215327842-1a4383c0-3296-4f8a-b7b2-c7946411335f.png)
 
  ~~~mysql
  select a.name, st_astext(a.polygon) from 28F_test.COUNTRY a where st_intersects
  (st_polygonFromtext(
  'POLYGON((125.71611639801966 35.138149565674794,
		        125.71611639801966 33.150938275638154,
            127.66691905038624 33.150938275638154,
		        127.66691905038624 35.138149565674794,
		        125.71611639801966 35.138149565674794))')
          , a.polygon);
  
  ~~~
  - Polygon, GeoMetry 타입들은 Binary로 저장되기 때문에 우리가 읽을 수 있는 String 타입으로 가져와야 눈으로 확인할 수 있다.
  - 그래서 `ST_ASTEXT`로 캐스트 해야한다.

  6. 해당 쿼리를 실행하여 `MBR` 과 폴리곤이 교차하는 국가,도시와 폴리곤을 아래와 같이 조회 할 수 있다.
  ![image](https://user-images.githubusercontent.com/79154652/215327966-f115e76a-2255-47d5-bbf7-7cdc8a3de62e.png)

  7. 하지만 쿼리 실행 속도를 보면 `0.765sec` 라는 실행 시간이 나온다. 이정도 쿼리 실행시간이면 필자의 프로젝트에서 사용하지 못 할 수준이다.
  ![image](https://user-images.githubusercontent.com/79154652/215328100-a30afdb2-3fdb-4b94-8564-b951fcc3ae7a.png)
  
  8. 인덱스를 타는지 확인해보기 위해 `explain`으로 확인해 보았다. 결과는 테이블 풀 스캔이었다. 분명 인덱스를 생성했는데 왜 인덱스를 타지않는지 한참동안 찾아보았고 이유는 위에서 먼저 설명했던 테이블을 생성시 SRID를 설정하지 않아서 해당 컬럼에 인덱스를 타지않았던 것이다.
![image](https://user-images.githubusercontent.com/79154652/215328180-842407f6-31d7-47fe-ae3a-338a35da955f.png)

 
 ## 0.765sec -> 0.015sec 로 개선하기
 
 - 위에서 테이블 풀스캔 했던 쿼리를 똑같이 실행하여 인덱스를 타게하여 쿼리 속도를 개선해 보겠다.


  1. 먼저 SRID가 설정되지 않은 `polygon`컬럼의 인덱스를 삭제하자.
  2. 테이블을 생성 혹은 수정하는 쿼리를 아래처럼 작성하자.
  ~~~
  create table country (
    id bigint auto_increment
    code ~~
    name ~~
    type ~~
    polygon polygon not null SRID 4326 혹은 SRID 0
    lod ~~
  )
  ~~~
  
 3. 다른 테이블 컬럼은 중요하지않아 `polygon` 만 작성해놓았다. 위에서 언급한 바와 같이 SRID 를 설정해주어야 하며, 글의 시작부분에 적어놓은 `GPS 의 기준이 되는 WGS84 시스템은 SRID 4326, 단순 직교 좌표계는 SRID 0 입니다.` 해당 설명에 대해 다시 한 번 언급한다.
 
 > 참고로 설정된 데이터를 INSERT 할 때 SRID 의 값을 다르게 저장하면 오류가 발생한다. SRID 4326으로 설정되어있는 테이블에 SRID 0을 INSERT
 ~~~
 insert into country (code, name, type, polygon, lod) values
 (1, paris, polygon, st_geomFromText('polygon((1 1, 4,4))', 0)
 ~~~
 > -- ERROR 3643 (HY000): The SRID of the geometry does not match the SRID of the column 'polygon'. The SRID of the geometry is 0, but the SRID of the column is 4326. Consider changing the SRID of the geometry or the SRID property of the column.


  4. SRID를 설정한 테이블을 생성하고 다시 `polygon`컬럼에 인덱스를 생성한다.
  5. 위와 동일한 쿼리를 실행한다.
   ~~~mysql
  select a.name, st_astext(a.polygon) from 28F_test.COUNTRY a where st_intersects
  (st_polygonFromtext(
  'POLYGON((125.71611639801966 35.138149565674794,
		        125.71611639801966 33.150938275638154,
            127.66691905038624 33.150938275638154,
		        127.66691905038624 35.138149565674794,
		        125.71611639801966 35.138149565674794))')
          , a.polygon);
  ~~~
  6. 결과는 동일할 것이기 때문에 실행 속도만 확인해 본다. `0.765sec -> 0.015sec'로 개선된 것을 확인할 수 있다.
  ![image](https://user-images.githubusercontent.com/79154652/215329563-95dcff85-a423-46b5-9b7d-2e0c727d07c2.png)
  
  7. 인덱스를 잘 타는걸로 확인된다.
  ![image](https://user-images.githubusercontent.com/79154652/215329781-ebe219a2-b490-4139-bf53-8bec8d872a58.png)



## 마무리

공간 인덱스가 타지않을 시 SRID를 잘 설정했는지 확인해보자. 

이제야 공간 인덱스를 적용해서 쿼리 성능을 개선하였다. 아직 프로젝트의 시작일 뿐이다.. 멀고도 먼 지구본 프로젝트에 대해 계속해서 공부하고 기술할 예정이다.


--참조--
[https://dev.mysql.com/doc/refman/5.7/en/using-spatial-indexes.html](https://dev.mysql.com/doc/refman/5.7/en/using-spatial-indexes.html)
[https://dev.mysql.com/doc/refman/8.0/en/spatial-index-optimization.html](https://dev.mysql.com/doc/refman/8.0/en/spatial-index-optimization.html)
[https://geojson.io/#new&map=6.63/34.072/125.756](https://geojson.io/#new&map=6.63/34.072/125.756)
