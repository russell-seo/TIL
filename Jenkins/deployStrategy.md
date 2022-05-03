  
  # 배포전략(Rolling, Canary, Blue/Green)
  
  - 최근에는 서비스를 작게 만들고(MSA) 더 자주 배포 하는 방식으로 진행되고 있다. 이러한 트렌드에 맞춰 서비스 배포 전략도 다양하게 개발되고 발전되어 왔다.
  - 본 포스트에는 가장 대표적인 배포 전략 Rolling, Canary, Blue/Grren 에 대해 기록해 볼려고 한다.



  ## Rolling
  
  Rolling 배포는 서버를 한대 씩 구 버전에서 새버전으로 교체해가는 전략이다. 서비스 중인 서버 한대를 제외시키고 그 자리에 새 버전의 서버를 추가한다. 이렇게 `구버전` 에서 `새버전`으로 트래픽을
  점진적으로 전환한다. 이와 같은 방식은 서버 수의 제약이 있을 경우 유용하나 배포 중 인스턴스의 수가 감소 되므로 서버 처리 용량을 미리 고려해야 한다.
  
  ![image](https://user-images.githubusercontent.com/79154652/165893668-f7ecc136-04ab-465d-aa1e-65869d495819.png)



  ## Blue/Green
  
  Blue/Green 배포는 구 버전에서 새버전으로 일제히 전환하는 전략이다. 구 버전의 서버와 새 버전의 서버들을 동시에 나란히 구성하고 배포 시점이 되면 트래픽을 일제히 전환 시킨다.
  하나의 버전만 프로덕션 되므로 버전관리 문제를 방지할 수 있고 또한 빠른 롤백이 가능하다. 또 다른 장점으로 운영 환경에 영향을 주지않고 실제 서비스 환경으로 새 버전 테스트가 가능하다.
  예를 들어 구 버전과 새 버전을 모두 구성하고 포트를 다르게 주거나 내부 트래픽일 경우 새 버전으로 접근하도록 설정하여 테스트를 진행해 볼 수 있다. 단 시스템 자원이 두배로 필요하고, 전체
  플랫폼에 대한 테스트가 진행되어야 한다.
  
  
  ![image](https://user-images.githubusercontent.com/79154652/165894608-403afc3e-eb9c-493d-8bd4-a573eeffe783.png)


  ## Canary
  
  `Canary`라는 용어의 어원을 알면 이해가 더 쉽다. Canary는 카나리아 라는 새를 일컫는 말인데, 이 새는 일산화탄소 및 유독가스에 매우 민감하다고 한다. 그래서 과거 광부들이
  이 새를 옆에두고 광산에서 일을 하다가 카나리아가 갑자기 죽게 되면 대피를 했다고 한다.
  Canary 배포는 카나리아 새처럼 위험을 감지할 수 있는 배포 기법이다. 구 버전의 서버와 새 버전의 서버들을 구성하고 일부 트래픽을 새 버전으로 분산하여 오류 여부를 판단한다.
  이 기법으로 A/B 테스트도 가능한데, 오류율 및 성능 모니터링에 유용하다. 트래픽을 분산시킬 라우팅은 랜덤으로 할 수도 있고 사용자 프로필등을 기반으로 분류할 수도 있다.
  분산 후 결과에 따라 새 버전이 운영 환경을 대체할 수 있고 다시 구 버전으로 돌아갈 수도 있다.
  
  ![image](https://user-images.githubusercontent.com/79154652/165894937-eaa1b093-f5d1-4656-8585-b664b59f7396.png)