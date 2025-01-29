---
{"dg-publish":true,"permalink":"/csci-699/paper-reviews/week-3/heckler/"}
---

# Heckler: Breaking Confidential VMs with Malicious Interrupts

Speaker: Aditya Rajan

# ENG

# **Heckler: Breaking Confidential VMs with Malicious Interrupts**  
**Speaker: Aditya Rajan**  

## **Key Summary: HECKLER - Attacking Confidential VMs**  

### **Overview**  
HECKLER is an attack where a hypervisor stealthily injects interrupts to hijack a program’s execution flow, thereby bypassing security authentication. *(No mitigation techniques proposed in the talk.)*  

When an interrupt occurs, the hypervisor **saves the current state of the VM** before handling or forwarding the interrupt. During this state-saving/restoration process, the hypervisor gains an opportunity to **manipulate register values**.  

---

## **Technical Techniques**  

### **1. Malicious Interrupt Injection**  
- The hypervisor injects interrupts that can modify the **registers and global state** of a Confidential VM (CVM) to manipulate its data and control flow.  
- The attack primarily exploits system call interrupts such as `int 0x80` to alter the behavior of specific applications within the CVM.  

### **2. Attack Examples**  
- **Bypassing authentication in OpenSSH and `sudo`** to gain root access.  
- **Tampering with statistical analysis software (C, Java, Julia)** to distort computation results.  

### **3. Step-by-Step Attack Chain**  
- Multiple injected interrupts are chained together to **gradually modify program state and control flow** over time.  

### **4. Vulnerabilities**  
- **AMD SEV-SNP and Intel TDX** forward certain interrupts to CVMs, which allows an attacker to exploit these interrupts to alter the state of a Confidential VM.  

---

## **Key Contributions**  

### **1. Introduction of a New Attack**  
- Demonstrates how **hypervisor-injected interrupts** can be exploited to manipulate data and control flow within a Confidential VM (CVM).  

### **2. Development of an Attack Chain**  
- Establishes a systematic technique to **chain injected interrupts together** to attack various services.  

### **3. Proof-of-Concept (PoC) Attack**  
- Provides experimental results showing successful authentication bypass in **AMD SEV-SNP and Intel TDX-based CVMs** (Confidential Virtual Machines).  
- Tests were conducted in multiple scenarios to demonstrate the real-world feasibility of HECKLER.  

---

## **Successful Attack Cases**  

1. **OpenSSH**  
   - Manipulated authentication logic to allow login **even with an incorrect password**.  

2. **sudo**  
   - Escalated **normal user privileges to root**.  

3. **Data Analysis Applications**  
   - Tampered with statistical analysis results, making the **data unreliable and untrustworthy**.  

---

## **Implications**  
- These attacks **break the confidentiality and integrity** guarantees of CVMs, demonstrating that they can be easily compromised through malicious interrupt manipulation.
---
# KOR
### 핵심 요약: HECKLER - Confidential VM 취약점 공격

#### 개요
HECKLER는 하이퍼바이저가 인터럽트를 몰래 집어넣어 프로그램의 실행 흐름을 가로채고, 이를 통해 보안 인증을 우회하는 것입니다. (해결 방식 제시 X)

인터럽트 발생할 때, 하이퍼바이저는 **현재 VM의 상태를 저장**한 후, 인터럽트를 처리하거나 전달합니다. 하이퍼바이저는 이 상태 저장/복원 과정에서 레지스터 값을 **조작**할 수 있는 기회를 얻습니다.

---

#### 기술적 기법
1. **악성 인터럽트 주입**:
   - 하이퍼바이저는 CVM의 레지스터와 글로벌 상태를 변경할 수 있는 인터럽트를 주입하여 프로그램의 데이터와 제어 흐름을 조작합니다.
   - 주로 `int 0x80`과 같은 시스템 콜 인터럽트를 활용하여 CVM 내 특정 응용 프로그램의 동작을 변조합니다.

2. **공격 사례**:
   - OpenSSH 및 `sudo`에서 인증을 우회하여 루트 접근 권한을 획득.
   - 통계 분석(C, Java, Julia)을 조작하여 계산 결과를 왜곡.

3. **단계적 공격 체인**:
   - 여러 인터럽트를 연속적으로 주입하여 프로그램 상태와 제어 흐름을 점진적으로 변형.

4. **취약점**:
   - AMD SEV-SNP와 Intel TDX는 일부 인터럽트를 CVM에 전달하며, 이로 인해 공격자가 인터럽트를 활용하여 CVM의 상태를 변경할 수 있습니다.

#### 주요 기여
- **새로운 공격 소개**:
  - 하이퍼바이저 주입 인터럽트를 악용해 CVM의 데이터와 제어 흐름을 변경.
- **공격 체인 개발**:
  - 주입된 인터럽트를 조합하여 다양한 서비스를 공격할 수 있는 체계적 기술 구축.
- **PoC (Proof-of-Concept) 공격**:
  - AMD SEV-SNP 및 Intel TDX 기반 CVM에서 OpenSSH와 sudo 인증을 우회하는 실험적 결과 제시.
HECKLER의 실제 가능성을 보여주기 위해 여러 상황에서 공격을 테스트했습니다.

- **성공한 사례**:
    1. OpenSSH:
        - 잘못된 비밀번호로도 로그인을 허가하도록 조작.
    2. sudo:
        - 일반 사용자 권한을 루트 사용자로 변경.
    3. 데이터 분석 애플리케이션:
        - 통계 분석 결과를 왜곡해 데이터를 신뢰할 수 없게 만듦.
- **의미**:
    
    - 이러한 공격은 CVM이 제공하는 **기밀성**과 **무결성**이 공격에 의해 쉽게 깨질 수 있음을 보여줍니다.
