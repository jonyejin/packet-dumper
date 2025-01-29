---
{"dg-publish":true,"permalink":"/csci-699/paper-reviews/week-3/td-xdown/"}
---

# TDXdown: Single-Stepping and Instruction Counting Attacks against Intel TDX
Speaker: Chenyu Zhou

# English

# TDXdown: Single-Stepping and Instruction Counting Attacks against Intel TDX  
**Speaker: Chenyu Zhou**  

## Intel Trust Domain Extensions (TDX)  
The vulnerabilities of **single-stepping detection**, a key defense mechanism of TDX, and the **instruction counting attack** based on it.  

### How Single-Stepping Works  

1. **Utilizing External Interrupts:**  
   - An attacker manipulates hardware timers (e.g., APIC timer) to trigger an interrupt at a specific point during program execution.  
   - When the interrupt occurs, the CPU halts the current execution and hands control over to the operating system (or hypervisor).  

2. **Executing Instructions One at a Time:**  
   - After the interrupt, the program resumes execution, but only **one instruction at a time**.  
   - By repeating this process, the attacker can **observe the execution flow at an instruction level**.  

3. **Using Side Channels:**  
   - The attacker analyzes memory access patterns, cache usage, power consumption, and other indicators that occur when a specific instruction is executed to extract sensitive data.  

### Attack Techniques  

- **Modifying CPU Frequency Settings:**  
   - The attacker lowers the CPU frequency of a specific core to its minimum (e.g., 800 MHz) using the **Linux cpufreq driver**.  
   - Disables CPU features like **Intel SpeedStep** to prevent dynamic frequency adjustments.  

- **Distorting `rdtsc` Timing:**  
   - The `rdtsc` (Read Time-Stamp Counter) instruction on modern Intel CPUs measures time using a **fixed frequency**.  
   - Lowering the CPU frequency slows down execution, but since `rdtsc` increments at a constant rate, the measured time appears normal.  
   - As a result, even if the execution of an instruction takes longer, the `rdtsc` value remains sufficiently large, making the delay appear as normal execution.  

- **TLB Flush:**  
   - To maximize the effects of single-stepping, the attacker flushes the **Translation Lookaside Buffer (TLB)**, artificially increasing the execution time of the first instruction.  

---

## What is a Trusted Domain (TD)?  

A **Trusted Domain (TD)** is an isolated execution environment created using Intel TDX to provide enhanced security by protecting the confidentiality and integrity of code and data from untrusted entities, including the hypervisor.  

## What is SEAM Mode (Secure Arbitration Mode)?  

SEAM Mode is a **dedicated CPU mode** required to run the **TDX module** and **Trusted Domains (TDs)**.  

- **VMX-root mode:**  
  - Runs the **TDX module**.  
  - The TDX module intermediates all interactions between TDs and the hypervisor and manages the highest level of SEAM mode.  

- **VMX-non-root mode:**  
  - Runs **Trusted Domains (TDs)**.  
  - Protects application code and data within the TD from external threats.  

SEAM Mode operates with **higher privileges** than traditional CPU modes (e.g., user mode, kernel mode), providing a security layer isolated from potential attackers such as hypervisors or operating systems.  

---

## **StumbleStepping Attack**  

- Exploits **Flush+Reload** cache attacks to extract information leaked by TDX’s single-stepping prevention mechanism.  
- Measures the number of executed instructions to infer control flow information of TDX-protected code.  

### **Why Does This Work?**  

- **Flush+Reload** is a cache side-channel attack technique that tracks **data access patterns** by detecting whether specific memory locations are cached.  

    1. The attacker **flushes** a specific memory area (e.g., TD’s memory) from the CPU cache.  
    2. Later, the attacker **reloads** the same memory location and measures the access time.  
    3. If the **data is in the cache**, access is fast; if not, access is slower, allowing the attacker to infer program execution states.  

- **Can this leak random numbers (nonces)?**  
  - Yes, by analyzing access patterns, attackers may deduce random values used in security protocols.  

---

## **Remaining Questions**  
(Questions for further discussion)  



Intel Trust Domain Extensions (TDX)
TDX의 주요 방어 기법인 **싱글 스테핑(single-stepping) 탐지**와 이에 기반한 **지시어 카운팅 공격(instruction counting attack)** 의 취약점

싱글 스테핑의 작동 방식
1. **외부 인터럽트 활용:**
    - 공격자는 하드웨어 타이머(예: APIC 타이머)를 조작하여 프로그램 실행 중 특정 지점에서 인터럽트를 발생시킵니다.
    - 인터럽트가 발생하면 CPU는 현재 실행 중인 작업을 멈추고 운영체제(혹은 하이퍼바이저)로 제어를 넘깁니다.
2. **명령어 단위 실행:**
    - 인터럽트가 발생한 후 다시 프로그램 실행을 재개하되, 한 번에 **한 명령어**만 실행하도록 제어합니다.
    - 이를 반복하면, 공격자는 프로그램의 실행 흐름을 **명령어 단위로 관찰**할 수 있습니다.
3. **부채널 활용:**
    - 특정 명령어가 실행될 때 발생하는 메모리 접근 패턴, 캐시 사용, 전력 소비 등의 정보를 분석하여 민감한 데이터를 유출합니다.

- **CPU 주파수 설정 변경:**  
    공격자는 **Linux cpufreq 드라이버**를 사용하여 특정 CPU 코어의 주파수를 최소 값(예: 800 MHz)으로 낮춥니다.
    - Intel SpeedStep과 같은 CPU 기능을 비활성화하여 CPU의 동적 주파수 조정을 방지합니다.
- **rdtsc 타이밍 왜곡:**  
    현대 Intel CPU의 `rdtsc`(Read Time-Stamp Counter) 명령어는 **고정 주파수**로 시간을 측정합니다. 주파수를 낮추면 CPU 실행 속도가 느려져도 `rdtsc` 카운터 값은 동일한 속도로 증가합니다.  
    결과적으로, 한 명령어를 실행하는 데 걸리는 시간이 늘어나도 `rdtsc` 값은 충분히 큰 값으로 기록되어 정상 실행으로 인식됩니다.
    
- **TLB 플러시(TLB Flush):**  
    Single-Stepping 효과를 극대화하기 위해 Translation Lookaside Buffer(TLB)를 플러시하여 첫 번째 명령어의 실행 시간을 인위적으로 늘립니다.

What is TD(Trusted Domain)?

What is SEAM Mode? (Secure Arbitration Mode)
- **TDX 모듈**과 **Trust Domain(TD)** 을 실행하는 데 필요한 전용 CPU 모드
	- VMX-root mode
		- **TDX 모듈**이 실행됩니다.
		- TDX 모듈은 TD와 하이퍼바이저 간의 모든 인터랙션을 중재하며, SEAM 모드의 최상위 계층을 관리합니다.
	- VMX-non-root mode
		- **Trust Domain(TD)** 가 실행됩니다.
		- TD 내부의 애플리케이션 코드와 데이터가 보호됩니다.
- 기존의 CPU 모드(예: 사용자 모드, 커널 모드)보다 높은 권한을 가지며, 특히 하이퍼바이저나 운영 체제(OS) 같은 공격 가능 주체로부터 격리된 보호 계층을 제공합니다.

**StumbleStepping 공격:**
- **Flush+Reload** 캐시 공격을 통해 TDX의 싱글 스테핑 방지 모드가 유출하는 정보를 활용.
- 실행된 명령어의 개수를 측정하여 TDX 보호 코드에 대한 제어 흐름 정보를 획득.

-> why???

- **Flush+Reload**는 캐시 부채널 공격 기법으로, 특정 메모리 주소가 캐시에 있는지 확인해 **데이터 접근 패턴**을 추적합니다.
    1. 공격자는 CPU 캐시에서 특정 메모리 영역(예: TD의 메모리)을 강제로 비우는(Flush) 작업을 합니다.
    2. 이후, 동일한 메모리 주소를 다시 접근해(Reload) 읽는 데 걸리는 시간을 측정합니다.
    3. **캐시에 데이터가 존재하면** 접근 시간이 짧고, 존재하지 않으면 길어지므로, 이를 통해 프로그램의 실행 상태를 파악할 수 있습니다.

-> 난수(nonce)를 알아낼 수 있다.

Remaining Questions

