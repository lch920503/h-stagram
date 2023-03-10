# 프로젝트 정보

- 소셜 네트워크 서비스 만들기

### 사용 기술

- axios를 이용해 데이터 fetching
- redux 사용하여 전역상태관리
- useState, useCallback, useSelector 등 다양한 hooks 사용
- firebase의 auth, database, storage 등 이용하여 데이터 구축 (express 사용)
- firebase로 호스팅 진행
- scss를 이용한 스타일링 진행
- react-router-dom v6 사용
- 피드에서 이미지 업로드 시, 로딩 스피너 사용

### 구현사항

[✔] 로그인 및 회원가입  
[✔] 메인피드  
[✔] 댓글 및 대댓글 작성  
[✔] 마이페이지  
[✔] 친구추천  
[✔] 팔로우 및 팔로잉  
[ ] 언팔하기  
[✔] 친구피드 보기

### 구현 코드 일부 - 서버

```javascript
// 회원가입

router.post("/user/new", (req, res) => {
  const { email, password, nickname } = req.body;

  firebaseAuth
    .createUser({
      email,
      password,
      displayName: nickname,
    })
    .then((credential) => {
      const { uid } = credential;

      Promise.all([
        // users/{uid}/profile/email,timestamp,nickname
        firebaseDatabase.ref(`/users/${uid}/profile`).set({
          email,
          nickname,
          timestamp: Date.now(),
        }),
        // 닉네임 중복 확인
        firebaseDatabase.ref(`/statics/nicknames/${uid}`).set(nickname),
      ])
        .then(() => {
          res.status(200).json({
            msg: "유저가 생성 되었습니다.",
          });
        })
        .catch((err) => {
          res.status(400).json({
            err,
          });
        });
    })
    .catch((err) => {
      res.status(400).json({
        err,
      });
    });
});
```

### 구현 코드 일부 - 컴포넌트

```javascript
import axios from "axios";
import React, { useCallback, useEffect, useState } from "react";
import { useSelector } from "react-redux";
import { useNavigate } from "react-router-dom";
import "./css/index.css";

export default function Join() {
  const navigate = useNavigate();

  const nicknames = useSelector((state) => state.config.service.nicknames);

  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [nickname, setNickname] = useState("");
  const [hasNickname, setHasNickname] = useState(false);

  const handleSubmitJoin = useCallback(
    (e) => {
      e.preventDefault();

      if (
        email &&
        password &&
        !hasNickname &&
        nickname &&
        password.length >= 6
      ) {
        axios({
          method: "post",
          url: "https://us-central1-h-stagram.cloudfunctions.net/clientApi/user/new",
          data: JSON.stringify({ email, password, nickname }),
          headers: {
            "Content-Type": "application/json",
            "Allow-Control-Access-Origin": "*",
          },
        })
          .then((res) => {
            navigate("/");
          })
          .catch((err) => console.log(err));
      } else {
        alert("올바르지 않은 값이 있습니다.");
      }
    },
    [email, password, nickname, hasNickname, navigate]
  );

  const checkNicknames = useCallback(() => {
    if (nicknames.includes(nickname)) {
      setHasNickname(true);
    } else {
      setHasNickname(false);
    }
  }, [nicknames, nickname]);

  useEffect(() => {
    checkNicknames();
  }, [checkNicknames]);

  return (
    <div className="join">
      <div className="wrapper">
        <h1 className="logo">
          <img src="/assets/welcome/logo.svg" alt="logo" />
        </h1>
        <form className="join-form" onSubmit={handleSubmitJoin}>
          <div className="email-input-box input-box">
            <label className="text-bold" htmlFor="email">
              이메일
            </label>
            <input
              type="email"
              id="email"
              placeholder="이메일을 입력해주세요."
              required
              autoComplete="on"
              onChange={(e) => setEmail(e.target.value)}
            />
          </div>
          <div className="password-input-box input-box">
            <div className="text-box">
              <label className="text-bold" htmlFor="password">
                비밀번호
              </label>
              <span className="helper-text">
                {password &&
                  password.length < 6 &&
                  "비밀번호는 6자리 이상 입력해주세요."}
              </span>
            </div>
            <input
              type="password"
              id="password"
              minLength="6"
              placeholder="비밀번호를 입력해주세요."
              required
              autoComplete="off"
              onChange={(e) => setPassword(e.target.value)}
            />
          </div>
          <div className="nickname-input-box input-box">
            <div className="text-box">
              <label className="text-bold" htmlFor="nickname">
                닉네임
              </label>
              <span className="helper-text">
                {hasNickname && "이미 사용중인 닉네임입니다."}
              </span>
            </div>
            <input
              type="text"
              id="nickname"
              placeholder="닉네임을 입력해주세요."
              required
              autoComplete="on"
              onChange={(e) => setNickname(e.target.value)}
            />
          </div>
          <button className="join-btn">회원가입하기</button>
        </form>
      </div>
    </div>
  );
}
```

### 구현화면

- 로그인
  ![login](https://user-images.githubusercontent.com/84822384/224247070-edbe9a57-391e-4741-ac69-3bd4e2c0c504.PNG)
- 회원가입
  닉네임 중복확인
  ![join](https://user-images.githubusercontent.com/84822384/224247061-bfd74e88-a5cf-4312-b803-a4f7f2cafc39.PNG)
- 메인피드
  텍스트 업로드, 이미지 업로드  
   내 피드 및 친구 피드 목록 보기  
   팔로잉 한 친구 목록 로드
  ![mainFeed](https://user-images.githubusercontent.com/84822384/224247072-b6c6750a-6ae4-4f60-9691-30ccd9b94e0a.PNG)
- 프로필
  추천 친구 피드로 이동  
   친구 팔로우 하기  
   본인 피드 리스트  
   추천 친구 기능 (회원가입 친구 목록 로드)
  ![profile](https://user-images.githubusercontent.com/84822384/224247080-8f515301-899b-4d8e-889d-643d8a2b439a.PNG)
