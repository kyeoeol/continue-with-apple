# Continue With Apple
Sign in with Apple, Sign up with Apple, or Continue with Apple

## Utils
### 1. randomNonceString
>로그인 요청마다 임의의 문자열인 'nonce'가 생성되며, 이 'nonce'는 앱의 인증 요청에 대한 응답으로 ID 토큰이 명시적으로 부여되었는지 확인하는 데 사용된다. 릴레이 공격을 방지하려면 이 단계가 필요하다.
><br>
>Adapted from https://auth0.com/docs/api-auth/tutorials/nonce#generate-a-cryptographically-random-nonce
```swift
func randomNonceString(length: Int = 32) -> String {
    precondition(length > 0)
    let charset: Array<Character> =
        Array("0123456789ABCDEFGHIJKLMNOPQRSTUVXYZabcdefghijklmnopqrstuvwxyz-._")
    var nonce = ""
    var remainingLength = length
    
    while remainingLength > 0 {
        let randoms: [UInt8] = (0 ..< 16).map { _ in
            var random: UInt8 = 0
            /// SecRandomCopyBytes(_:_:_)를 사용하여 암호로 보호된 'nonce' 를 생성한다.
            let errorCode = SecRandomCopyBytes(kSecRandomDefault, 1, &random)
            if errorCode != errSecSuccess {
                fatalError("Unable to generate nonce. SecRandomCopyBytes failed with OSStatus \(errorCode)")
            }
            return random
        }
        
        randoms.forEach { random in
            if remainingLength == 0 {
                return
            }
            
            if random < charset.count {
                nonce.append(charset[Int(random)])
                remainingLength -= 1
            }
        }
    }
    
    return nonce
}
```

### 2. Secure Hashing Algorithm
>256비트 다이제스트를 사용한 SHA-2(Secure Hashing Algorithm 2) 해싱 구현
><br>
>hashing ; 해싱은 하나의 문자열을 원래의 것을 상징하는 더 짧은 길이의 값이나 키로 변환하는 것
><br>
>암호용 해시 함수는 매핑된 해싱 값만을 알아가지고는 원래 입력 값을 알아내기 힘들다는 사실에 의해 사용될 수 있다.
```swift
func sha256(_ input: String) -> String {
    let inputData = Data(input.utf8)
    let hashedData = SHA256.hash(data: inputData)
    let hashString = hashedData.compactMap {
        return String(format: "%02x", $0)
    }.joined()
    
    return hashString
}
```

## Delegate
>인증에 성공하면 ASAuthorizationController는 app이 사용자 데이터를 키 체인에 저장하는 데 사용하는 함수를 호출
><br>
>사용자 정보(이메일, 이름 등)는 초기 등록 시에만 ASAuthorizationAppleIDCredential로 전송되고 이후에는 userIdentifier만 전송된다.
```swift
func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization)
```

## Credential
### 1. appleIDCredential
```swift
/// userIdentifier
let userIdentifier = appleIDCredential.user

/// A JSON Web Token (JWT) that securely communicates information about the user to the app.
let identityToken = appleIDCredential.identityToken

/// short-lived token as proof that it has authorization to interact with the server
let authorizationCode = appleIDCredential.authorizationCode
```

### 2. passwordCredential
>기존 iCloud 키체인 자격 증명을 사용하여 로그인
```swift
let user = passwordCredential.user
let password = passwordCredential.password
```
