# 소셜 네트워크 서비스 만들기

### 구현사항

[✔] 로그인 및 회원가입  
[✔] 메인피드  
[✔] 마이페이지  
[✔] 친구추천  
[✔] 팔로우 및 팔로잉  
[ ] 언팔하기  
[✔] 친구피드

### 사용 기술

- axios를 이용해 데이터 fetching
- redux 사용하여 상태관리
- useCallback, useSelector 등 다양한 hooks 사용
- firebase의 auth, database, storage 등 이용하여 데이터 구축
- firebase로 호스팅 진행
- scss를 이용한 스타일링 진행
- react-router-dom v6 사용

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
