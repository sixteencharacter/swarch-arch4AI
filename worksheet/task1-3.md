# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** อมร พันธุรัตน์

**Student ID / รหัสนิสิต:** 6632250621

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 507 | 593 | 93.5% | 100% |
| t0-small | 16 | 37 | 72.5% | 100% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> Large model ให้ accuracy เพิ่ม แต่แลกมาด้วย latency ที่มากขึ้นทั้งสำหรับผู้ใช้ทั่วไปและผู้ใช้ที่โชคร้าย อย่างมีนัยสำคัญ

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 27.0 | 230 | 392 | 18 | 0 | 94.5% |
| t1-after | 9.5 | 402 | 1012 | 0 | 24 | 92.6% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> ทำให้เข้าใจผิดเนื่องจากว่า latency น้อยกว่าความเป็นจริงของ baseline แต่ column ที่บอกความจริงก็คือ errors จะสังเกตได้ว่าเกิด error มาก หลาย transction จึงตัดก่อนเวลาที่ควรจะใช้จริง ๆ

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> request ที่รออยู่อาจจะถือ CPU time ใน thread และ memory บางส่วนไว้ สำหรับการรัน business logic และการ routing ในทั้ง review service และ gateway เนื่องจากว่ารอ inference service ในการ inference ให้เสร็จ

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> ควรจะจัดการโดยการนำมาประมวลผลใหม่ในภายหลัง โดยอาจจะใช้ topology แบบ dataflow ที่ทำงานตามช่วงเวลาที่กำหนดในการเปลี่ยนรูปแบบของ data เหล่านี้จนเป็น data ที่ถูกต้องในที่สุด

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> ถ้าตั้งเป็น 0.1 วินาที สิ่งที่เกิดขึ้นคือเราอาจจะไม่สามารถได้รับคำตอบทันจากการประมวลผลจาก AI เนื่องจากว่า AI ปกติจะใช้เวลานานในการประมวลผล ซึ่งการตั้ง timeout เป็น 0.1 นี่เองจะทำให้ข้อมูลที่จริง ๆ ควรจะประมวลผลได้ จะได้ผลว่าเป็น error แบบ false positive แทน แต่ถ้าหากตั้งไว้ที่ 10 วินาที สิ่งที่เกิดขึ้นคือระบบจะใช้เวลามากขึ้นในการรอการประมวลผลจาก AI ซึ่งหากการประมวลผลนั้นเกิด error ระบบก็จะเสียเวลารอไปแบบฟรี ๆ

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 8 | 553 | 91.5% | 38% |
| t2-0.7 | 0.7 | 385 | 571 | 93.0% | 53% |
| t2-0.8 | 0.8 | 441 | 580 | 93.5% | 64% |
| t2-0.9 | 0.9 | 484 | 601 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> จะ deploy CampusEats ที่ threshold = 0.6 เพราะว่าการเพิ่ม threshold ทำให้ accuracy เพิ่มได้อย่างมากเพียง 2.5% จากเดิมเท่านั้น เมื่อใช้ช่วงความมั่นใจที่ 0.6-0.9 ดังนั้นจึงใช้ค่า threshold นี้เพราะว่าสามารถทำ accuracy ได้ที่ 91% อยู่แล้ว

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> เนื่องจากว่า p50 จะดูค่าเฉลี่ยของ response latency ของผู้ใช้งานทั้งระบบ แต่ว่า p95 จะดูผู้ใช้งานที่โชคร้าย (เป็น 5% ของ request ที่ช่าที่สุดในระบบ) ซึ่งการใช้ large model จะทำให้เกิดการใช้ latency มาก ดังนั้น p95 จึงมักจะสะท้อนในส่วนของ response time ของ request ที่ใช้โมเดลขนาดใหญ่เหล่านั้น แทนที่จะเป็น request ทั่วไปส่วนใหญ่ที่ไม่ได้ใช่โมเดลส่วนใหญ่ ดังนั้นการลด threshold จึงช่วย p50 ได้มากกว่า p95

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> เหตุการณ์ที่สามารถเกิดขึ้นได้ก็คือในกรณีที่ small model ถูก train มาด้วยข้อมูลที่มีความ biased ซึ่งการมี bias นั้นจะทำให้ค่าความมั่นใจของโมเดลผิดพลาด และ ส่งผลให้คำตอบเหล่านั้นไม่ได้ถูกประมวลผลอย่างลึกซึ้งโดยโมเดลที่ใหญ่กว่าได้ ซึ่งสิ่งนี้ก็จะส่งผลให้ผู้ใช้งานได้รับข้อมูลที่ผิดพลาดจากความ bias ของโมเดลขนาดเล็กไป

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 7 | 541 | 93.0% | 38% | 35% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> รีวิวที่ได้รับประโยชน์จาก cache คือรีวิวที่มีใจความสั้น ๆ และใช้คำไม่แตกต่างกันมากนัก เช่น superb! กับ superb! แต่หากเป็นประโยคยาว ๆ ที่สามารถเกิดความหลากหลายของการใช้คำได้ ประโยคเหล่านั้นไม่มีทางจะได้ประโยชน์จาก cache นี้ เนื่องจากว่าการทำ cache ในตอนนี้เป็นการ check แบบ exact match

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> ต้อง invalidate cache ทั้งหมดเนื่องจากว่าโมเดลได้เกิดการเปลี่ยนแปลงไปแล้ว แต่ในทางที่ดีแล้วควรที่จะทำ TTL ของ cache แต่ละตัวเพื่อที่จะให้สามารถได้ข้อมูลที่สดใหม่ตามระยะเวลาที่กำหนด ดังนั้นเราสามารถออกแบบ TTL ของ cache เพิ่มเติมเพื่อที่จะทำให้การ invalidate และ update cache สามารถเกิดขึ้นโดยอัตโนมัติได้

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> สิ่งที่ได้เรียนรู้ที่ไม่มีใน lecture ก็คือเราสามารถใช้งาน OrderedDict มาทำ cache อย่างง่ายได้ ซึ่งเป็นสิ่งที่ไม่ได้มีการกล่าวถึงในช่วง lecture
