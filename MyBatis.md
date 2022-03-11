
# MyBatis Join 


  MyBatis를 사용한 Join 한 데이터를 DTO에 매핑하는 내용을 살펴볼 것이다.
  
  필자는 ResultMap을 활용하여 안정적으로 매핑을 하였다.
  
  다음은 PushInfoVO 에 매핑한 예시 이다.
  
  __PushInfoVO__
  ~~~java
  @Data
public class PushInfoVO {

    private String id;
    private String tradesub_nm;
    private String login_time;
}
  
  ~~~
  
  __Mapper.xml__
  
  ~~~java
  <resultMap id="PushInfoUser" type="com.jhc.logen.vo.fcm.PushInfoVO">
        <id column="id" property="id"/>
        <result column="tradesub_nm" property="tradesub_nm" />
        <result column="login_time" property="login_time"/>
    </resultMap>
  
  ~~~
  
  - `<resultMap>` 은 하나의 매핑 정보를 작성한다.
      - `type`은 매핑 결과가 담길 DTO 타입이다. -> 위의 예제는 PushInfoVO에 해당 값을 매핑한다.
      - `id`는 resultMap을 구별하는 이름으로 Unique 해야 한다.
  - `<id>` 는 Primary Key에 해당하는 컬럼을 설정한다.
      - `column = "id"` 는 테이블의 컬럼 명을 적는다. -> DB의 컬럼명이 tradesub_cd 이면 column = "tradesub_cd" 라고 설정한다.
      - `property = "id"` 는 조회 결과가 매핑될 DTO의 이름이다. -> 위 PushInfoVO의 변수가 `id`로 설정되어 있어 `id`로 설정.
  - `<result>` 는 PK가 아닌 일반 컬럼에 대한 매핑을 처리한다. 속성은 위의 `<id>` 와 동일하다. 
  
  
  ~~~java
  <select id="findUserInfo" parameterType="String" resultMap="PushInfoUser">
        select l.id, a.tradesub_cd, a.tradesub_nm, a.login_time from
                      LOGEN_APP.logen_fcm l
                          left join
                          applog.app_use_login a
                              on l.id = a.tradesub_cd
                                where l.tel = #{tel} and l.isPush = 'O'
    </select>
  
  ~~~
  
  
  - `resultMap`을 사용할 때는 returnType 대신 resultMap 속성에 사용하려는 resultMap의 `id`값을 넘겨 주면 된다.
  - 위의 resultMap 에 `id` 값이 `PushInfoUser`로 적용되어 있어, `resultMap = PushInfoUser` 로 적용해주면 된다.
  
  
  __Mapper Interface__
  
  ~~~java
  public interface FcmInfoMapper {
    
    List<PushInfoVO> findUserInfo(PushInfoRequestVO tel_no);
  }
  
  ~~~
