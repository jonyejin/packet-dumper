---
{"dg-publish":true,"permalink":"/dns-sec-domain-name-system-security-extensions/"}
---

### DNSSEC (Domain Name System Security Extensions)
DNSSEC is a protocol extension designed to enhance the security of the DNS by protecting against data spoofing and tampering. It uses cryptographic methods to ensure data integrity, origin authentication, and secure denial of existence.

![](https://miro.medium.com/v2/resize:fit:1400/1*GLQ4YeaOBFvV6u-sfhm17Q.png)


![](https://miro.medium.com/v2/resize:fit:930/1*oCW3Ed9Ylb1MkFtgW9o-uw.png)


DNSSEC(Domain Name System Security Extensions)은 DNS(Domain Name System)의 보안을 강화하기 위한 확장 프로토콜입니다. DNS는 도메인 이름을 IP 주소로 변환하는 중요한 인터넷 서비스인데, 기본적으로 보안 메커니즘이 없어 데이터 위조나 변조에 취약합니다. DNSSEC은 이런 문제를 해결하고자 설계되었습니다.

DNSSEC은 기존 DNS에 다음과 같은 새로운 레코드를 추가합니다:
- **RRSIG**: 리소스 레코드의 디지털 서명.
- **DNSKEY**: 공개 키를 포함한 레코드.
- **DS**: 부모 영역에 저장되는 해시 정보. 상위 도메인에 저장된 하위 도메인의 키 서머리. DNSKEY의 hash값
- **NSEC/NSEC3**: 존재하지 않는 도메인 이름 증명.
- **CDNSKEY/CDS**: 키 전송 자동화를 위한 레코드.

- RRset: 레코드를 private key로 sign함.
- RRSIG: RRset에서 나온? signature을 RRSIG record에 보관한다.
- DNSKEY record: 다른 사람들이 내꺼 해독할 수 있게 private key 뿌림.

DNS서버에 A record 요청시
- A record와 RRSIG 같이 리턴. 
- DNSKEY도 별도로 요청해서 받아야함.

하지만 attacker가 DNSKEY를 MITM로 Spoofing해버리면 그만임.  -> Key-Signing Key로 DNSKEY의 유효성도 입증해야 한다.

DNSKEY 받을 때 RRSIG를 하나 더 받는다. 이 signature는 zsk와 ksk를 모두 포함한다.

Chain of Trust
 - Whenever a child zone is setup, a hashed copy of the zone’s Public key-signing key (KSK) is given to the parent zone to publish as a new type of record called the **Delegation Signer (DS) record**.
- How this works is that each time a resolver is told to contact a child zone, the parent zone provides the resolver with a DS record. 
- Not only does this tell the resolver that the child zone supports DNSSEC (DNSSEC is far from ubiquitous), but it also provides a way of validating and trusting the child zone.
- All the DNS resolver needs to do, is hash the public KSK of the child zone and compare it to the DS record provided by the parent. If the records match up, then the resolver can be assured that it is talking to the child-zone’s authorized DNS server.
- This also tells the resolver that the child’s KSK has not been tampered with, and that all records in the child zone can be trusted.

- The next obvious question is okay so now we can trust the child zone, but how do we establish trust of the parent zone and its DS record. This is where the chain of Trust and Private root-signing key come into play.

How to trust the root???
- They do Key signing event.
- They announce to the world??


1. 데이터 무결성 보장  
   DNSSEC은 디지털 서명을 통해 DNS 응답이 변경되지 않았음을 보장합니다.

2. 출처 인증  
   클라이언트가 DNS 응답이 신뢰할 수 있는 권한 서버에서 왔는지 확인할 수 있습니다.

3. 거부 응답 보호  
   존재하지 않는 도메인에 대한 응답도 서명을 통해 보장하여 DNS 응답 위조 공격(예: DNS 스푸핑)을 방지합니다.

 장점
- 데이터 위조 및 변조 방지
- 중간자 공격(MITM) 방지
- DNS 스푸핑 및 캐시 포이즈닝 공격 방어
 단점
- 설정 및 관리 복잡성
- 성능 저하(추가 데이터 전송 및 계산 필요)
- 완전한 보안 보장을 위해 DNSSEC을 전 세계적으로 채택해야 함

 요약
DNSSEC은 인터넷 보안을 한층 강화하는 기술이지만, 설정의 복잡성과 인프라 요구사항으로 인해 아직 모든 도메인에 보편적으로 사용되지는 않고 있습니다. 하지만 점점 더 많은 도메인이 DNSSEC을 채택하고 있습니다. 


 동작 원리

DNSSEC는 공개 키 기반 암호화(Public Key Cryptography)를 사용하여 DNS 레코드의 무결성을 보장하고, 클라이언트가 응답의 출처를 검증할 수 있게 합니다. 다음은 DNSSEC가 동작하는 과정을 단계별로 설명합니다:

---

 1. DNS 쿼리 발생
클라이언트(예: 웹 브라우저)가 특정 도메인 이름에 대한 IP 주소를 요청합니다.

---

 2. DNSSEC 검증 활성화
클라이언트가 DNSSEC 검증 기능을 지원하는 리졸버(DNS Resolver)를 사용 중이라면, 리졸버는 도메인에 대한 DNSSEC 데이터를 요청합니다.

 3. 서명된 응답 전달
DNSSEC가 활성화된 DNS 서버는 요청된 DNS 리소스 레코드(Resource Record, 예: A, MX, NS 레코드)와 함께 서명된 데이터(RRSIG 레코드)를 반환합니다. 이 RRSIG 레코드는 해당 레코드에 대한 디지털 서명을 포함합니다.

---

 4. 공개 키를 통한 검증
클라이언트의 리졸버는 응답된 데이터를 검증하기 위해 DNSKEY 레코드를 요청합니다.  
- DNSKEY에는 해당 도메인의 공개 키가 포함되어 있습니다.
- 리졸버는 이 공개 키를 사용하여 RRSIG의 서명을 검증하고 데이터의 무결성을 확인합니다.

---

 5. DNSKEY 검증
DNSKEY 자체도 서명되어 있으며, 상위 도메인(예: `.com` 또는 `.kr`)에 있는 DS 레코드를 사용하여 검증됩니다.  
- DS 레코드는 부모 영역에서 자식 도메인의 DNSKEY를 해싱한 값입니다.
- 리졸버는 상위 도메인의 DS 레코드와 하위 도메인의 DNSKEY를 비교하여 유효성을 확인합니다.

---

 6. 신뢰 체인 (Chain of Trust)
DNSSEC는 최상위 루트 도메인(`.`)부터 하위 도메인까지 신뢰 체인을 구축합니다.
- 루트 도메인의 공개 키가 신뢰할 수 있는 키로 미리 설정되어 있습니다(전역적으로 배포된 루트 키).
- 리졸버는 루트 키를 시작으로 단계별로 하위 도메인(예: `.com`, `example.com`)의 키를 검증하며 내려갑니다.

---

 7. 데이터 무결성 및 출처 검증
리졸버는:
- DNS 레코드가 디지털 서명된 데이터와 일치하는지 확인(데이터 무결성).
- 응답이 권한 있는 DNS 서버에서 온 것인지 확인(출처 인증).

만약 검증에 실패하면, 클라이언트는 응답을 무시하고 오류를 반환합니다.

---

 8. 거부 응답 보호
존재하지 않는 도메인 이름에 대한 요청이 발생하면, DNS 서버는 NSEC/NSEC3 레코드를 반환하여 특정 도메인이 없음을 증명합니다.
- 이 레코드도 서명되어 있어 위조가 불가능합니다.

---

 예제: DNSSEC 응답의 흐름
1. 클라이언트가 `example.com`의 IP 주소를 요청.
2. DNSSEC 지원 리졸버가 `example.com`의 A 레코드와 RRSIG 레코드를 요청.
3. DNS 서버가 A 레코드와 RRSIG 레코드를 반환.
4. 리졸버는 `example.com`의 DNSKEY를 요청하여 서명을 검증.
5. DNSKEY는 상위 `.com` 영역의 DS 레코드를 통해 검증.
6. `.com`의 DNSKEY는 루트 도메인의 DS 레코드를 통해 검증.

---

 요약
1. DNSSEC는 루트에서 시작하여 하위 도메인으로 내려가는 신뢰 체인을 통해 데이터를 검증합니다.
2. 공개 키 암호화를 사용하여 디지털 서명 검증을 수행합니다.
3. 데이터가 변조되지 않았음을 보장하며, 출처를 신뢰할 수 있도록 합니다.

 DANE?

DANE(DNS-based Authentication of Named Entities)는 DNSSEC를 기반으로 한 인증 프로토콜로, X.509 인증서를 DNS 레코드에 저장하고 검증하는 방식으로 동작합니다. 전통적인 인증서 신뢰 체계인 CA(Certificate Authority)에 의존하지 않고, DNSSEC의 신뢰 체인을 활용해 인증서를 검증합니다.

 DANE의 주요 목적

1. 인증서 검증 방식 개선:  
    DANE은 SSL/TLS 인증서를 CA(Certificate Authority)만 신뢰하지 않고 DNSSEC를 통해 인증할 수 있도록 지원합니다.
    
2. CA의 의존성 감소:  
    전통적인 신뢰 체계에서 발생할 수 있는 CA의 위조 및 오작동 문제를 방지합니다.
    
3. 신뢰 체계 강화:  
    DNSSEC를 사용하여 인증서와 관련된 데이터를 안전하게 전송하고 검증합니다.
    

---

 DANE의 핵심 요소

1. TLSA 레코드:  
    DANE은 새로운 DNS 레코드인 TLSA(Transport Layer Security Authentication) 레코드를 사용합니다.  
    TLSA 레코드는 특정 도메인 이름에 대해 사용할 인증서를 지정하고, 이를 검증하는 데 필요한 정보를 제공합니다.
    
    - 형식: `_port._protocol.domain IN TLSA usage selector matching_type certificate_association_data`
    - 예: `_443._tcp.example.com. IN TLSA 3 1 1 {hash}`
2. DNSSEC 사용:  
    DANE은 DNSSEC를 필수로 사용하여 TLSA 레코드의 무결성과 신뢰성을 보장합니다.
    

---

 TLSA 레코드의 필드 구성

TLSA 레코드는 다음과 같은 필드로 구성됩니다:

1. Usage: 인증서 사용 목적을 정의.
    
    - 0: CA가 발급한 인증서 체인을 검증.
    - 1: 특정 CA의 인증서만 허용.
    - 2: 특정 인증서만 허용.
    - 3: PKI(CA) 체계를 사용하지 않고 자체 인증서만 허용.
2. Selector: TLSA 레코드가 참조하는 인증서 데이터의 유형.
    
    - 0: 전체 인증서를 사용.
    - 1: 인증서의 주체 공개 키만 사용.
3. Matching Type: 인증서 데이터를 비교할 때 사용할 해시 알고리즘.
    
    - 0: 데이터 원본 그대로 사용.
    - 1: SHA-256 사용.
    - 2: SHA-512 사용.
4. Certificate Association Data: 지정된 인증서 또는 공개 키에 대한 해시 값.
    

---

 DANE의 작동 과정

1. 클라이언트가 특정 도메인(예: `example.com`)에 HTTPS 연결을 요청합니다.
2. 클라이언트는 DNSSEC를 통해 해당 도메인의 TLSA 레코드를 조회합니다.
3. TLSA 레코드에서 제공된 정보로 서버의 인증서를 검증합니다.
    - 인증서가 TLSA 레코드와 일치하면 신뢰합니다.
    - 일치하지 않으면 연결을 거부합니다.

---

 DANE의 장점

1. 보안 강화:  
    DNSSEC의 신뢰 체인을 통해 TLSA 레코드를 안전하게 보호하며, CA를 통한 인증서 위조를 방지합니다.
    
2. 유연성:  
    자체 서명 인증서(self-signed certificates)를 허용하면서도 안전하게 사용할 수 있습니다.
    
3. CA 의존도 감소:  
    전통적인 CA 신뢰 체계를 보완하거나 완전히 대체할 수 있습니다.
    

---

 DANE의 사용 사례

1. SMTP 보안 강화:  
    메일 서버 간의 TLS 연결에 DANE을 활용하여 중간자 공격(MITM)을 방지.
    
2. HTTPS 인증서 관리:  
    DANE을 통해 HTTPS 서비스의 인증서를 관리하고, CA로부터의 의존도를 줄임.
    
3. IoT 및 자체 서명 인증서 환경:  
    CA 없이도 안전하게 통신할 수 있는 IoT 환경에 적합.
    

---

 DANE의 한계

1. DNSSEC 의존성:  
    DANE은 DNSSEC가 활성화된 환경에서만 동작합니다. 아직 전 세계적으로 DNSSEC의 도입률이 낮습니다.
    
2. 기존 생태계와의 호환성:  
    기존 브라우저와 애플리케이션이 DANE을 지원하지 않는 경우가 많습니다.
    
3. 복잡성 증가:  
    DNSSEC 및 DANE 설정과 관리가 어렵고, 추가적인 리소스가 필요합니다.
    

---

DANE은 기존의 CA 중심 인증서 관리 방식을 보완하거나 대체할 수 있는 강력한 방법으로, 특히 DNSSEC가 활성화된 환경에서 높은 보안성을 제공합니다.


 PKI vs DANE


