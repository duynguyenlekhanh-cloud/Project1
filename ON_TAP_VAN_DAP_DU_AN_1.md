# TÀI LIỆU ÔN TẬP VẤN ĐÁP DỰ ÁN 1 — TÌM KIẾM (PAC-MAN AI)
**Môn học:** Trí tuệ nhân tạo (AI)  
**Điểm autograder hiện tại:** 26/25 điểm

---

## I. TỔNG QUAN 4 THUẬT TOÁN TÌM KIẾM (file `search.py`)

Cả 4 thuật toán đều là **Graph Search** (tìm kiếm đồ thị có tập `visited` để tránh lặp vô tận). Điểm khác biệt nằm ở cấu trúc dữ liệu của `frontier`:

| Thuật toán | Cấu trúc dữ liệu (`frontier`) | Tiêu chí chọn nút mở | Tối ưu (Optimal)? |
| :--- | :--- | :--- | :--- |
| **Q1: DFS** | `util.Stack` (LIFO) | Đi sâu nhất trước (nút mới vào sau) | **Không** |
| **Q2: BFS** | `util.Queue` (FIFO) | Đi nông nhất trước (nút vào trước ra trước) | **Có** (nếu mọi bước đi bằng chi phí) |
| **Q3: UCS** | `util.PriorityQueue` | Chi phí tích lũy $g(n)$ nhỏ nhất | **Có** (mọi bước đi $\ge 0$) |
| **Q4: A\*** | `util.PriorityQueue` | Tổng $f(n) = g(n) + h(n)$ nhỏ nhất | **Có** (nếu $h$ admissible / consistent) |

---

## II. BÍ KÍP TRẢ LỜI CÁC CÂU HỎI VẤN ĐÁP THẦY CÔ HAY HỎI NHẤT

### Câu 1: DFS (`depthFirstSearch`)
* **Hỏi:** "Tại sao DFS không tìm ra đường đi ngắn nhất?"
* **Trả lời:** Vì DFS cứ thấy nhánh nào là đâm sâu vào nhánh đó đến tận cùng. Nó có thể tìm thấy một đích ở rất sâu trước khi kịp xét một đích ở ngay sát nút gốc.
* **Hỏi:** "Tại sao trong frontier lại lưu tuple `(state, actions)`?"
* **Trả lời:** Để khi gặp đích (`isGoalState`), ta trả về ngay danh sách hành động `actions` mà không cần mảng truy vết `parent`.

---

### Câu 2: BFS (`breadthFirstSearch`)
* **Hỏi:** "BFS có luôn tối ưu không?"
* **Trả lời:** BFS chỉ tối ưu khi chi phí các bước đi là **như nhau** (ví dụ mỗi bước đều tốn 1 đơn vị). Nếu các bước có chi phí khác nhau, BFS không đảm bảo tối ưu.
* **Hỏi:** "Độ phức tạp bộ nhớ của BFS so với DFS?"
* **Trả lời:** BFS tốn bộ nhớ hơn rất nhiều ($O(b^d)$) vì phải lưu toàn bộ các nút ở tầng hiện tại. DFS chỉ tốn $O(b \cdot m)$ với $b$ là hệ số nhánh, $m$ là độ sâu tối đa.

---

### Câu 3: UCS (`uniformCostSearch`)
* **Hỏi (RẤT QUAN TRỌNG):** "Tại sao phải kiểm tra `isGoalState` lúc POP khỏi hàng đợi mà không kiểm tra lúc PUSH?"
* **Trả lời:** Vì lúc PUSH một nút vào frontier, ta mới chỉ tìm thấy **một** đường đi tới nó. Có thể tồn tại đường đi khác vòng vèo hơn nhưng tổng chi phí lại rẻ hơn và được duyệt sau. Chỉ khi nút được **POP** ra khỏi PriorityQueue, ta mới chắc chắn rằng đường đi tới nó là rẻ nhất!

---

### Câu 4: A\* (`aStarSearch`)
* **Hỏi:** "Nếu hàm heuristic $h(n) = 0$ ở mọi nơi thì A\* trở thành thuật toán gì?"
* **Trả lời:** Trở thành UCS (Uniform Cost Search).
* **Hỏi:** "Heuristic Admissible (chấp nhận được) và Consistent (nhất quán) là gì?"
* **Trả lời:** 
  - **Admissible:** $h(n) \le h^*(n)$ (không bao giờ ước lượng vượt quá chi phí thực tế).
  - **Consistent:** $h(A) \le cost(A, B) + h(B)$ (thỏa mãn bất đẳng thức tam giác).

---

### Câu 5: Bốn góc (`CornersProblem` trong `searchAgents.py`)
* **Hỏi:** "Biểu diễn trạng thái (`State`) của bài toán 4 góc gồm những gì?"
* **Trả lời:** Gồm một tuple 2 phần tử: `((x, y), (bool1, bool2, bool3, bool4))`
  - `(x, y)`: Vị trí hiện tại của Pac-Man.
  - Tuple 4 boolean: Trạng thái đã ghé thăm của 4 góc. Dùng `tuple` thay vì `list` vì tuple là kiểu dữ liệu bất biến (hashable), đưa được vào tập `visited = set()`.
* **Hỏi:** "Tại sao không dùng toàn bộ `GameState` làm trạng thái tìm kiếm?"
* **Trả lời:** Vì `GameState` chứa rất nhiều thông tin dư thừa (vị trí ma, điểm số, các hạt thức ăn khác...). Dùng `GameState` sẽ gây bùng nổ không gian trạng thái, làm tràn bộ nhớ và chạy cực chậm.

---

### Câu 6: Heuristic 4 góc (`cornersHeuristic`)
* **Hỏi:** "Heuristic này tính như thế nào? Có admissible không?"
* **Trả lời:** Từ vị trí Pac-Man, tham lam chọn góc chưa ghé thăm gần nhất (theo khoảng cách Manhattan), cộng dồn khoảng cách, nhảy tới góc đó rồi lặp lại cho tới khi hết góc. Manhattan distance luôn $\le$ khoảng cách thực tế trong mê cung nên luôn admissible và consistent.

---

### Câu 7: Ăn hết thức ăn (`foodHeuristic`)
* **Hỏi:** "Tại sao lấy khoảng cách tới hạt xa nhất (`max(mazeDistance)`) mà vẫn admissible?"
* **Trả lời:** Để ăn hết tất cả thức ăn, Pac-Man bắt buộc phải đi tới hạt xa nhất. Do đó chi phí thực tế tối thiểu chắc chắn phải $\ge$ khoảng cách tới hạt xa nhất. Vì vậy $h(n) \le h^*(n)$ (luôn admissible).
* **Hỏi:** "`problem.heuristicInfo` dùng để làm gì?"
* **Trả lời:** Dùng để lưu bộ nhớ đệm (caching / memoization) các khoảng cách mê cung đã tính, tránh tính lại hàm `mazeDistance` nhiều lần gây chạy quá thời gian (hết giờ).

---

### Câu 8: Điểm thức ăn gần nhất (`findPathToClosestDot` & `AnyFoodSearchProblem`)
* **Hỏi:** "Tại sao lại dùng BFS trong `findPathToClosestDot`?"
* **Trả lời:** Vì BFS duyệt theo từng lớp khoảng cách tăng dần, nên hạt thức ăn đầu tiên mà BFS chạm tới chắc chắn là hạt **gần nhất**.
* **Hỏi:** "Chiến lược tham lam (cứ ăn hạt gần nhất) có tối ưu toàn cục không? Cho ví dụ phản chứng?"
* **Trả lời:** **Không tối ưu.** Ví dụ: Pac-Man ở giữa, bên trái có 1 hạt cách 2 bước, bên phải có cụm 10 hạt cách 3 bước. Tham lam sẽ chạy sang trái ăn 1 hạt, sau đó phải lộn ngược toàn bộ quãng đường chạy sang phải ăn 10 hạt kia $\rightarrow$ tổng quãng đường dài hơn rất nhiều so với việc đi sang phải trước.

---

## III. VỀ VẤN ĐỀ TRÙNG LẶP CODE TRÊN MẠNG
- Thuật toán DFS, BFS, UCS, A\* là các thuật toán kinh điển từ sách giáo khoa (Russell & Norvig), cấu trúc chuẩn chỉ có 1 cách viết tối ưu nhất.
- Thầy cô đưa ra cảnh báo đó chủ yếu để **phỏng vấn/vấn đáp trực tiếp**. Chỉ cần bạn hiểu bản chất từng dòng code và trả lời được các câu hỏi ở phần II thì 100% đạt điểm tối đa!
