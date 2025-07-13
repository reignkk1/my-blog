---
title: "styled-components의 함정과 최적화"
category: "architecture"
date: "2025-07-06"
desc: "부드러운 재생바를 만들다 뚝뚝 끊기는 현상을 해결한 이야기입니다."
thumbnail: "./images/styled-components의 함정과 최적화 이야기/thum.jpg"
alt: "ui/ux"
---

음악 플레이어를 만들면서 사용자 경험을 더 향상시키기 위해 재생바(progress bar)가 부드럽고 자연스럽게 움직이도록 구현하고자 했습니다.
이를 위해 음악의 현재 재생 시간을 소수점 단위(예: 12.3532초, 12.4212초...)로 실시간 추적하고 이 값을 기반으로 재생바의 `background`색상이 점진적으로 채워지게 만들었습니다.

</br>
<img src='./images/styled-components의 함정과 최적화 이야기/code.png'/>
</br>
</br>

위 코드처럼 `props.progress` 값을 사용해 `linear-gradient`의 퍼센트 위치를 동적으로 설정하면 재생 시간이 증가함에 따라 재생바의 색상이 자연스럽게 채워지는 시각 효과를 낼 수 있습니다.

처음에는 모든 게 정상적으로 작동하는 것처럼 보였지만 음악을 조금만 재생해보니 재생바가 간헐적으로 끊기며 뚝뚝 끊어지는 현상이 발생했습니다.

</br>
<img style="display: block; margin: 0 auto;" src='./images/styled-components의 함정과 최적화 이야기/Animation.gif'/>
</br>
</br>

문제를 추적해본 결과 'styled-components'의 클래스 생성 방식 때문이었습니다.

## 문제의 원인: 클래스 네임 생성 & 캐싱 누적

`styled-components`는 `props`가 변경될 때마다 내부적으로 새로운 CSS 클래스 네임을 생성하고, 해당 스타일을 메모리에 캐싱하는 방식으로 동작합니다.

그런데 재생바에 사용된 `progress` 값은 음악 재생 중 실시간으로 그것도 소수점 단위까지 끊임없이 변하는 값입니다.

즉 재생이 진행되는 매 순간마다 `props`가 갱신되며 그에 따라 `styled-components`는 매번 새로운 클래스 네임을 생성하고 저장합니다.

</br>
<img style="display: block; margin: 0 auto;" src='./images/styled-components의 함정과 최적화 이야기/html_1.gif'>
</br>
</br>

이러한 과정이 반복되면서 렌더링 시 브라우저가 처리해야 할 클래스가 점점 많아지고 결국에는 렌더링 병목이 발생해 재생바가 끊기는 현상으로 이어졌습니다.

## 해결 방법: 인라인 스타일로 전환

이 문제를 해결하기 위해 저는 styled-components 대신 인라인 스타일 방식을 선택했습니다.

인라인 스타일은 별도의 클래스를 생성하거나 캐시하지 않고 해당 DOM 요소의 style 속성에 직접 값을 적용하기 때문에 렌더링 비용이 훨씬 적습니다.

</br>
<img style="display: block; margin: 0 auto;" src='./images/styled-components의 함정과 최적화 이야기/animation_2.gif'>
</br>
</br>

위 GIF에서 볼 수 있듯이 기존에는 클래스가 계속 바뀌면서 렌더링이 끊기던 재생바가
인라인 스타일로 바꾼 후에는 부드럽게 자연스럽게 동작하는 모습을 확인할 수 있습니다.

</br>
<img style="display: block; margin: 0 auto;" src='./images/styled-components의 함정과 최적화 이야기/html_2.gif'>
</br>
</br>

인라인 스타일은 단순히 값을 바로 바꿔주는 방식이라 매번 새로운 클래스 생성이나 캐시 관리 작업이 필요하지 않기 때문에 성능적으로 매우 효율적입니다. 특히 프레임 단위로 props가 바뀌는 UI에 적합합니다.

## 결론: 언제 styled-components를 피해야 할까?

실시간으로 값이 자주 그리고 미세하게 변하는 UI에서는 styled-components의 클래스 생성 방식이 오히려 성능 저하의 원인이 될 수 있습니다. 이런 경우에는 인라인 스타일이 훨씬 가볍고 효율적인 대안이 됩니다.

styled-components는 강력하고 선언적인 CSS-in-JS 도구이지만 모든 상황에서 최선의 선택은 아니라는 것을 이 경험을 통해 체감하게 되었습니다.
