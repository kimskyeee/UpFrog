# 올라락! 깨구락!
올라락! 깨구락!은 언리얼엔진으로 개발된 멀티플레이 플랫포머 게임으로, 게임 내 UI와 장애물을 구현했습니다.

🎥 아래의 사진을 클릭시 게임 플레이 요약 영상을 보실 수 있습니다.

[![Video Label](http://img.youtube.com/vi/KTFeGCObKgA/0.jpg)](https://youtu.be/KTFeGCObKgA)

<br>

## 사용 기술
- Unreal Engine 5.5
- C++ / Blueprint 혼합 구조
- Rider

<br>

## 프로젝트 개요
| 항목 | 내용 |
| --- | --- |
| **장르** | 수직 플랫포머 게임 |
| **플랫폼** | PC (Steam 연동) |
| **맵 컨셉** | 두 가지 방식을 지원합니다. <br>1. 맵 에디팅을 통해 직접 맵을 생성후 플레이 <br>2. 스토리를 따라 스테이지형 맵을 플레이 |
| **개발 내용** | 게임 장애물 개발, 친구목록 등 UI, 게임 진행도 UI, 음성퀴즈 (AI 협업, WebSocket) |

<br>

## 구현 상세 설명
각 기능의 더 자세한 설명은 [여기](https://www.notion.so/kimskye/1ceb7f13b7a68181a1e4ddf2c48ec842)서 확인하실 수 있습니다.

---

## 1. UI / UX 개발
게임의 전반적인 UI 및 UX 개발을 담당했습니다.

### 친구목록 및 방 목록 UI
- **친구목록 UI**: Stream API를 활용해 현재 로그인 중인 친구 목록을 표시할 수 있습니다.
- **방 목록 UI**: 멀티플레이 시 참여할 수 있는 세션 목록을 보여줍니다.

<table>
  <tr>
    <td><img src="https://github.com/user-attachments/assets/2f3a8128-9253-431b-8056-724854b2176f" height="350"/></td>
    <td><img src="https://github.com/user-attachments/assets/9a4c7f68-ba51-4e5e-94b4-27d8a80ed459" height="350"/></td>
  </tr>
</table>

- 각 목록 UI에는 오브젝트 풀링을 적용하여, 반복 생성 없는 UI 위젯 재활용 구조로, 메모리 낭비 없이 성능을 극대화했습니다.
  
<br>

### 화면 전환 카메라 컴포넌트
메인로비, 스테이지, 커스텀 게임 화면 등 화면 시점을 다르게 보여주는 연출에서 **UI간 순환참조를 방지하기 위해** 카메라 컴포넌트를 구현하였습니다.

https://github.com/user-attachments/assets/e2e332b5-0571-42dd-a1c2-c799f0d95590

<br>

### 게임 진행도 UI
멀티플레이시 게임 시작 지점과 끝 지점의 z좌표 위치를 받아와 **1등 플레이어를 기준**으로 전체 진행률을 계산하는 UI를 구현하였습니다.

https://github.com/user-attachments/assets/ae8eb996-4b8d-41cc-937d-5c7464245817

<br>

---

## 2. 게임 장애물 개발
게임 내 다양한 오브젝트(장애물, 장식물, 건축물 등)의 공통 기반 클래스인 `ABaseProp`을 설계한 뒤, 하위 클래스인 `AObstacleProp`(장애물) 로직을 구현하였습니다.

### 장애물 목록

| 이름 | 구현 방식 요약 |
| --- | --- |
| `ABouceBollardProp` | `LaunchCharacter`를 통해 **캐릭터 진행 방향의 반대로** 튕겨내는 막대 구현 |
| `ARotateHammerProp` | `LaunchCharacter`를 통해 **Prop의 UpVector 방향**으로 튕겨내는 망치 구현 |
| `ARotatePlatformProp` | `AddLocalRotation`을 통해 **회전하는 플랫폼**(바닥)을 구현 |
| `ATrampolineProp` | `LaunchCharacter`를 통해 **Prop의 UpVector 방향**으로 튕겨내는 트램펄린 구현 |
| `ACannonBallProp` | `LaunchCharacter`를 통해 **Prop의 ForwardVector 방향**으로 튕겨내는 대포알 구현 |
| `AConveyorBeltProp` | `AddActorWorldOffset`을 통해 플레이어를 **가속/감속** |

### 장애물 구현 영상

https://github.com/user-attachments/assets/bfd4d5bf-eb1d-4641-8f47-e7aac7c1e5af

<br>

### `LaunchCharacter` 동기화
- 게임 내 존재하는 대포, 트램펄린, 튕김 막대 등 대다수의 장애물이 `LaunchCharacter`로 구현되었습니다.
- 멀티플레이에서 **장애물 충돌 동기화**를 위해 **클라이언트 예측 구조**를 사용하였습니다.
  - 기존 방식: 서버가 충돌 감지 → `Multicast`로 튕김 반응 적용 (문제: 서버·클라이언트 위치 불일치 시 충돌 누락)
  - 개선 방식: 클라이언트가 먼저 `LaunchCharacter()` 로컬 실행 → 서버에 RPC 요청으로 권한 확정 → 서버가 `Multicast`로 전체 동기화

### 대포 장애물: 오브젝트 풀링
- 대포가 일정 시간마다 대포알을 발사하는 장애물에서, 오브젝트 풀링(Object Pooling)을 적용해 대포알을 재사용했습니다.
<br>

---

## 3. AI 협업 WebSocket기반 음성퀴즈
- AI 팀과의 협업으로, 플레이어의 대답을 기반으로 유사도를 판단해 통과 시 플레이어에게 버프를 부여하는 장애물을 구현했습니다.

https://github.com/user-attachments/assets/520105b5-faf4-477d-a215-d4aabc7c546d
  
- 플레이어의 음성 입력을 실시간으로 녹음 후, **AI 서버와 WebSocket**을 통해 주고받으며 음성 유사도를 판별해 퀴즈 결과를 처리하는 시스템을 설계했습니다.

<img width="1642" height="528" alt="스크린샷 2025-06-14 202214" src="https://github.com/user-attachments/assets/14d089ab-43ed-44a8-971a-6d5637d2bc1e" />

---

<br>

## 트러블 슈팅
- **음성 퀴즈 시스템의 청크 사이즈 및 패딩 문제** 를 해결했습니다.
- 프로젝트 관련 트러블 슈팅은 [여기](https://www.notion.so/kimskye/1ceb7f13b7a68181a1e4ddf2c48ec842?source=copy_link#209b7f13b7a68060a840e85122028c76)에서 확인하실 수 있습니다.

