# nginx-stream-manager-with-bird
가 아니라 BGP 아이피 애니캐스트로 배포하려고 하는데 어차피 서버에 다 bird든 뭐든 넣을거고
적당히 시간 지날때마다 깃 풀 받아서 인터페이스도 먹이고 nginx도 reload 시키면 되겠다 싶어서 만든
stream 매니저가 아니라 interface + stream proxy config generator가 적합할 것 같은데
일단 한 번 만들었으니 이걸 쓴다면 애니캐스트는 정말 불가능한게 아님을 실감하게 될터이니, 이 프로그램을
한 번 써보기를 바란다고 생각하기 보다는 혼자 쓰려고 만들었지만 그래도 이거 쓰고 싶어하는 사람이 있을 수 있으니
내가 친히 매뉴얼을 작성해주겠음.

## 제대로 된 설명
- Bird 등을 통해서 아이피를 라우팅 하는 경우, 보통은 와이어가드나 OpenVPN 혹은 GRE 등을 써서 하게 될텐데, 솔직히 방화벽 관련 문제도 있고 번거롭잖아요?
- 그냥 외부 아이피 그대로 쓰면 센시스랑 쇼단에 돌아다니더라고요. 아우. 이게 막는다고 막히는게 아니야.
- 그럼 내부 아이피만 할당하고 외부 아이피로 인바운드만 엔진엑스가 스트림으로 적당히 처리해주면 안전한거 아냐? 라는 이상한 아이디어가 떠올라서 그대로 아무 생각 없이 만들어보았습니다.
- 그럼 아웃바운드는 어찌 처리하냐구요? 알아서 잘 처리하면 됩니다. 제가 또 이거 어느순간 아웃바운드 족도 구현해서 올려줄지 모르잖아요.
- 단순 인바운드 처리 + 애니캐스트 지원을 위해서 만든거지 그 이상도 이하도 아니라는 점...
- Bird 설정이나 nginx 설정은 알아서 하면 됩니다. nginx는 인클루드 한 번, cron도 적당히 수정해주세요.
- 혼자 쓰려고 만든거라 대충 대충 만들었음.

## v6 지원
- 생각도 시도도 안해봤습니다.
- 아마 [v6아이피] 이런식으로 감싸서 하면 될거라고 생각합니다.

## 쓰는 방법
1. generate.js 파일 수정하기
```js
const IPs = [
    {
        sourceProtocol: 'tcp',
        sourceIP: '10.120.120.1',
        sourcePort: '25565',
        destinationProtocol: 'tcp',
        destinationIP: '211.211.211.211',
        destinationPort: '25565',
        proxyControl: false,
    },
];
```

이 부분을 알아서 수정하시오.

아마 똑똑한 여러분이라면 이게 뭐하는건지는 실행시켜보고 알게 될 것.

2. prefixList 수정
```js
const prefixList = [
    '10.120.120.'
];
```

이 부분은 어나운스한 아이피 대역으로 적당히 수정해서 사용할 것.

## 자동화
crontab
```text
*/1 * * * *      git pull git@github.com/쌸라쌸라 && tunnroute.sh
```

대충 저렇게 처리하고 nginx reload 시키는 구절 추가.

## 파일 아웃풋
1. stream.conf
```
# auto-generated - 0
server {
    listen 10.120.120.1:25565;
    proxy_bind 10.120.120.1;
    proxy_pass 211.211.211.211:25565;
}

# (...)
```

2. tunnroute.sh
```sh
ip link add dev tunnroute type dummy
ip link set tunnroute up
ip link del dev tunnroute 10.120.120.0/32
ip link add dev tunnroute 10.120.120.1/32
ip link del dev tunnroute 10.120.120.2/32
# (...)
ip link del dev tunnroute 10.120.120.254/32
```

## 기여하기
- 포크 떠서 여러분의 프로젝트로 등록하세요.
- 저는 이 프로젝트 자체가 너무 완벽해서 따로 유지관리를 하지 않을 것 같습니다.

## 스크립트 라이선스
- 이 스크립트의 라이선스는 [Hakurei Reimu Public License](https://github.com/Alex4386/HRPL) (원본), [하쿠레이 레이무 공개 라이선스](https://github.com/HanbitGaram/HRPL) (번역본) 에 따라 공개됩니다.
- **책임의 부인** : 이 코드를 동방 프로젝트의 몇 캐릭터들이나 로봇,개, 혹성탈출 또는 인간 등이 사용해서 발생하는 문제점은 스크립트 제작자는 절대로 책임을 지지 않습니다.