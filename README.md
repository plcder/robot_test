# 機器人關節控制抗電磁干擾(EMI)通訊架構
# Robot Joint Control EMI-Resilient Communication Architecture

一套結合互斥對編碼(Dual-Rail / Mutual Exclusion Encoding)與擴展漢明碼(Extended Hamming Code, SECDED)的分層抗干擾架構,設計目標為機器人關節控制訊號在電磁干擾(EMI)環境下的可靠傳輸。

A layered, EMI-resilient communication architecture for robot joint control, combining dual-rail (mutual exclusion) encoding with Extended Hamming Code (SECDED). Designed for reliable signal transmission in electromagnetically noisy environments.

---

## 📄 文件 / Documents

- [技術文件(Markdown)/ Technical Document (Markdown)](./機器人抗EMI通訊架構技術文件.md)
- [技術文件(HTML,含目錄導覽)/ Technical Document (HTML, with navigation)](./robot-emi-architecture.html)

---

## 🇹🇼 專案簡介

### 問題背景

機器人手臂的關節控制訊號,在實際運作環境中會受到伺服馬達 PWM 雜訊、鄰近變頻器輻射、線材串擾等電磁干擾(EMI)影響,可能導致控制位元被意外翻轉,使機器人執行非既定軌跡,造成安全風險。

### 這套架構做了什麼

本專案推導並記錄一套**兩層防禦架構**:

1. **傳輸層 — Extended Hamming(8,4)**:自動糾正單一位元錯誤,並在偵測到雙重位元錯誤時誠實觸發安全停止,而非靜默送出錯誤指令(SECDED, Single Error Correction Double Error Detection)
2. **致動層 — 互斥對編碼**:透過接線拓樸本身,從物理層級排除「矛盾動作同時觸發」的可能性,不依賴軟體邏輯是否正確

完整推導包含最小漢明距離的數學證明、Hamming(7,4) 到 Extended Hamming(8,4) 的構造過程,以及與工業步進馬達驅動序列的對應關係。

### ⚠️ 誠實的範疇說明(請務必先讀這段)

- 本架構設計目標是**電磁干擾(EMI)**——持續性、中低強度、通常只影響少數位元的雜訊。**不適用於電磁脈衝(EMP)**等瞬間高能量、大規模同時破壞硬體的威脅。
- 本架構**不提升機器人的自主決策能力或智慧**,解決的是訊號完整性問題,與感知、規劃、AI 決策層次的問題完全無關。
- 架構中的每個組成元件(dual-rail encoding、Hamming code、SECDED、defense-in-depth 分層防禦原則)**皆為既有成熟理論**,並非本專案首創。本專案的定位是:**將這些分散在不同工程領域的既有技術,系統性整合並完整推導,具體應用於機器人關節控制場景**。經檢索,目前未發現有文獻明確以此具體組合方式應用於此場景。
- 尚未經過實體硬體原型驗證,目前僅為理論推導與架構設計,實際部署前建議進行原型測試與安全認證評估。

### 適用場景

| 適合 | 不適合 |
|---|---|
| 高電磁干擾工業環境(電焊機器人、電力設備旁) | 一般辦公室/家用機器人 |
| 安全關鍵應用(醫療手術機器人、人機協作機器人) | 成本敏感、大量生產的消費型機器人 |
| 長距離訊號傳輸的大型機械手臂 | 短距離、晶片內部訊號 |

### 後續方向

- [ ] 使用 Arduino/樹莓派 + 步進馬達 + 訊號產生器進行雜訊注入原型測試
- [ ] 量化實際部署環境的 EMI 強度,評估成本效益
- [ ] 探討三重以上位元錯誤的處理策略(如 BCH code)
- [ ] 與 IEC 61800-5-2、ISO 13849 等工業安全標準對照分析

---

## 🇺🇸 Project Overview

### Background

Robot arm joint control signals are exposed to electromagnetic interference (EMI) from servo motor PWM noise, nearby inverters, and cable crosstalk during real-world operation. Bit flips caused by such interference can lead robots to execute unintended trajectories, creating safety risks.

### What This Architecture Does

This project derives and documents a **two-layer defense architecture**:

1. **Transmission Layer — Extended Hamming(8,4)**: Automatically corrects single-bit errors, and honestly triggers a safe-stop when a double-bit error is detected, instead of silently issuing an incorrect command (SECDED — Single Error Correction, Double Error Detection).
2. **Actuation Layer — Dual-Rail (Mutual Exclusion) Encoding**: Physically eliminates the possibility of contradictory actuator commands firing simultaneously, at the wiring-topology level — independent of whether the software logic is correct.

The full derivation includes a mathematical proof of the minimum Hamming distance guarantee, the construction from Hamming(7,4) to Extended Hamming(8,4), and its correspondence to industry-standard stepper motor drive sequences.

### ⚠️ Honest Scope Disclosure (please read before using)

- This architecture targets **EMI (electromagnetic interference)** — continuous or intermittent, low-to-moderate intensity noise typically affecting only a few bits at a time. It is **not suitable for EMP (electromagnetic pulse)** threats, which involve instantaneous, high-energy, large-scale simultaneous damage, potentially including physical hardware destruction.
- This architecture **does not improve a robot's autonomous decision-making or intelligence**. It solely addresses signal integrity and has no relation to perception, planning, or AI decision-making layers.
- Every component in this architecture (dual-rail encoding, Hamming codes, SECDED, defense-in-depth layering) is an **existing, well-established theory**, not an original invention of this project. The contribution here is the **systematic integration and complete derivation of these techniques — drawn from otherwise separate engineering domains — applied specifically to robot joint control**. No literature was found during research that explicitly applies this specific combination to this specific use case.
- This has **not yet been validated on physical hardware**. It is currently a theoretical derivation and architectural design only. Prototype testing and safety certification are recommended before real-world deployment.

### Suitable Use Cases

| Suitable | Not Suitable |
|---|---|
| High-EMI industrial environments (welding robots, near power equipment) | General office/home robots |
| Safety-critical applications (surgical robots, human-robot collaboration) | Cost-sensitive, mass-produced consumer robots |
| Large industrial arms with long signal transmission lines | Short-distance, on-chip signal paths |

### Roadmap

- [ ] Prototype validation using Arduino/Raspberry Pi + stepper motor + signal generator noise injection
- [ ] Quantify actual deployment-environment EMI intensity to assess cost-effectiveness
- [ ] Explore handling strategies for 3+ simultaneous bit errors (e.g., BCH codes)
- [ ] Cross-reference against industrial safety standards such as IEC 61800-5-2 and ISO 13849

---

## 📚 相關既有理論 / Related Prior Art

| 組成元件 / Component | 對應理論 / Prior Theory | 起源 / Origin |
|---|---|---|
| 互斥對編碼 / Dual-rail encoding | m-out-of-n code | 非同步電路設計 / Asynchronous circuit design, 1980s– |
| 距離與糾錯關係 / Distance–error-correction relation | Hamming code theory | Richard Hamming, 1950 |
| SECDED | Extended Hamming code | 業界標準(伺服器 ECC 記憶體)/ Industry standard (server ECC memory) |
| 分層防禦 / Defense in depth | — | 廣泛見於量子糾錯、工業安全、資安 / Widely used in quantum error correction, industrial safety, cybersecurity |
| 循環驅動序列 / Cyclic drive sequence | Stepper motor full-step sequence | 電機工程標準 / Standard motor engineering practice |

---

## 📜 授權 / License

尚未指定,建議依發布目的選擇(如 MIT、CC-BY-4.0)。
Not yet specified — choose based on your intended use (e.g., MIT for code, CC-BY-4.0 for documentation).

## ✍️ 作者說明 / Author's Note

本文件記錄一套完整、邏輯自洽的推導過程,誠實標註其與既有理論的關係,尚未經過硬體原型驗證。歡迎工程社群提出意見與批評指正。

This document records a complete, logically self-consistent derivation process, with honest attribution to prior theory. It has not yet been validated on hardware. Feedback and critique from the engineering community are welcome.
