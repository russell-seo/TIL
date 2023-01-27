# Spatial Query Index 최적화



  ## SRID(Spatial Reference Identifier)
  
  공간 인스턴스에는 SRID(spatial reference identifier)가 있습니다. SRID는 평면 지구 매핑 또는 둥근 지구 매핑에 사용되는 특정 타원면을 기준으로 하는 공간 참조 시스템에 해당합니다.

  공간 열은 SRID가 다른 개체를 포함할 수 있습니다. 그러나 데이터에 대한 SQL Server 공간 데이터 메서드를 사용하여 작업을 수행할 때는 SRID가 동일한 공간 인스턴스만 사용할 수 있습니다. 

  두 공간 데이터 인스턴스에서 파생되는 모든 공간 메서드의 결과는 이러한 인스턴스에 동일한 SRID가 있을 경우에만 유효합니다. 이 SRID는 인스턴스의 좌표를 결정하는 데 사용되는 동일한 측정 단위, 데이터 및 프로젝션을 기준으로 합니다. 가장 일반적인 SRID의 측정 단위는 미터 또는 제곱미터입니다.

  두 공간 인스턴스의 SRID가 같지 않을 경우 이 인스턴스에서 사용되는 geometry 또는 geography 데이터 형식 메서드의 결과로 NULL이 반환됩니다. 예를 들어 NULL이 아닌 결과를 반환하는 다음 조건자 조건의 경우 및 의 두 geometry1 geometry geometry2인스턴스의 SRID가 같아야 합니다.
  
  `GPS 의 기준이 되는 WGS84 시스템은 SRID 4326, 단순 직교 좌표계는 SRID 0 입니다.`
