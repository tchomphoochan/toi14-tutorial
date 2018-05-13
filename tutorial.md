# Table of Contents

**1. Graph Algorithms**
  - 1.1 Graph Representation
    - 1.1.1 Adjacency Matrix
    - 1.1.2 Adjacency List
    - 1.1.3 Edge List
  - 1.2 Graph Traversal
    - 1.2.1 ใช้ Depth-first Search หา cycle ใน undirected graph
    - 1.2.2 ใช้ DFS หา cycle ใน directed graph
    - 1.2.3 ใช้ DFS เพื่อนับจำนวน component ใน undirected graph
    - 1.2.4 ใช้ BFS หาเส้นทางสั้นสุดใน unweighted graph
    - 1.2.5 ใช้ DFS เพื่อเช็คความเป็น bipartite graph
  - 1.3 Shortest Path Algorithm
    - 1.3.1 Dijkstra’s Algorithm
    - 1.3.2 Bellman-Ford Algorithm
    - 1.3.3 Shortest Path Faster Algorithm
    - 1.3.4 Floyd-Warshall Algorithm
    - 1.3.5 ปัญหาประยุกต์ Shortest Path
  - 1.4 Minimum Spanning Tree
    - 1.4.1 Kruskal's Algorithm
    - 1.4.2 Prim's Algorithm
    - 1.4.3 ปัญหาประยุกต์ MST
  - 1.5 Topological Sorting
  - 1.6 Bridges and Articulation Point
    - 1.6.1 นิยามเกี่ยวกับ edge ใน Depth-first Search
    - 1.6.2 นิยาม Discovery Time และ Lowlink Value
    - 1.6.3 คุณสมบัติของ Articulation Point
    - 1.6.4 คุณสมบัติของ Bridge
    - 1.6.5 โค้ดของ Tarjan's Algorithm

# 1. Graph Algorithms

## 1.1 Graph Representation

### 1.1.1 Adjacency Matrix

ถ้ารับ edge `(u, v)` มาก็แค่เซตให้ `Matrix[u][v] = 1` (ถ้าเป็น undirected graph ก็ `Matrix[v][u] = 1` ด้วย)

### 1.1.2 Adjacency List

สำหรับ Adjacency List ให้ใช้ array of vectors เอา แบบนี้

```cpp
vector<int> G[num_of_nodes];
// เพิ่ม edge (u,v) => G[u].push_back(v);
```

เวลาต้องการหาทุก node ที่อยู่รอบ ๆ node u ใด ๆ สามารถใช้การลูปแบบนี้ได้
```cpp
for (auto v : G[u]) {
    // อยากทำอะไรก็เชิญ
}
```

ถ้าเป็นกราฟที่มีน้ำหนัก แนะนำให้ใช้เป็น pair<int, int> เอา โดยที่ตัวแรกเก็บ node ปลายทาง ส่วนตัวที่สองเก็บน้ำหนัก แบบนี้

```cpp
using pii = pair<int, int>; // หรือจะ typedef ก็แล้วแต่สะดวก
vector<pii> G[num_of_nodes];
// เพิ่ม edge (u,v,w) => G[u].push_back({v, w});
// หรือจะใช้เป็น G[u].emplace_back(v, w); ก็ได้
```

### 1.1.3 Edge List

ถ้ารับ edge `(u, v)` มา ก็ยัดเป็น `pair<int, int>` ใส่ `vector` ไว้ได้เลย หรือถ้ากราฟมีน้ำหนัก อาจจะประกาศ `struct` ขึ้นมาก่อน หรือใช้ `pair<pair<int, int>, int>` ก็ได้

## 1.2 Graph Traversal

หลัก ๆ ก็มี Depth-first Search กับ Breadth-first Search นอกจากนี้ ยังต้องนำมาประยุกต์เป็นอีกด้วย ยกตัวอย่างการประยุกต์ดังนี้

### 1.2.1 ใช้ DFS หา cycle ใน undirected graph

หลักการคร่าว ๆ คือ เราจะ DFS ไปเรื่อย ๆ แล้วจด node ที่เคยเจอไว้ ถ้าเดินทางไป ๆ มา ๆ วนกลับมา node เดิมที่เคย visit แล้วก็ถือว่าเจอ cycle
ต้องระวังไว้สองอย่างหลัก ๆ คือ 1) ต้องเก็บ parent ไว้ตลอด เพราะเวลา dfs เราอาจจะเผลอเดินจาก node `u` ไป `v` แล้วเดินจาก `v` กลับมา `u` ซ้ำผ่าน edge เดิม และ 2) กราฟอาจจะ disconnect ออกจากกันได้ ดังนั้น ต้องลูปทดลองให้ครบ มั่นใจว่าทดลองทุก node แล้ว

```cpp
bool visited[N]; // เริ่มมาเซตทุกอย่างเป็น false

// dfs จะ return true ถ้าเจอ cycle, false ถ้าไม่เจอ cycle
// p คือ parent, ต้องเก็บไว้ด้วย กันเดิน edge เดิมซ้ำ
bool dfs(int u, int p) {
    if (visited[u]) // เจอ cycle แล้ว
        return true;
    visited[u] = true;
    for (auto v : G[u]) {
        if (v == p) // skip ข้าม edge parent ไป
            continue;
        if (dfs(v, u)) // ถ้า dfs ไปแล้วพบว่าเจอ cycle ก็ให้ return true ตาม
            return true;
    }
    return false; // ถ้าทำจนหมดแล้วยังไม่เจอ ก็ตอบ false
}

// ใน main:
bool found_cycle = false;
for (int u = 1; u <= n; ++u) {
    if (!visited[u]) {
        if (dfs(u)) {
            found_cycle = true;
            break;
        }
    }
}
// ต้องลูปทดลอง node ที่ยังไม่เคยไป เพราะกราฟอาจจะแยกจากกันได้
```

### 1.2.2 ใช้ DFS หา cycle ใน directed graph

สำหรับ Directed Graph จะต่างออกไป เราใช้ `visited` เฉย ๆ ไม่ได้แล้ว แต่จะใช้ `color` แทน โดยแทนสถานะดังนี้
- **สีขาว (color 0)** แทน node ที่ยังไม่ได้เริ่ม visit แม้แต่นิดเดียว
- **สีเทา (color 1)** แทน node ที่ visit แล้ว และกำลังอยู่ระหว่างการ DFS ตัวลูก ๆ อยู่
- **สีดำ (color 2)** แทน node ที่ visit แล้ว และจัดการ DFS ตัวลูก ๆ ครบหมดแล้ว

ตอนแรกทุก node เป็นสีขาว และเมื่อ DFS ตอนแรกเราจะเซต node นั้นเป็นสีเทา แล้วทำการ DFS ตัวที่อยู่ติดกันทั้งหมด เมื่อทำครบแล้วจึงเซต node นี้เป็นสีดำ ถ้าเผลอเดินทางไป node ใดที่เป็นสีเทาอยู่ซ้ำ ถือว่าเจอ cycle

เหตุผลที่ต้องใช้สี เพราะต้องกันกรณีเคสแบบนี้: `2 --> 1 --> 3` สังเกตว่าเคสนี้ ถ้าเราใช้ `visited` array แล้วเริ่ม DFS จาก node 1 ก่อน จะทำให้ `visited[1] = visited[3] = true` และเมื่อเรามาลอง DFS node 2 ต่อ เราจะมาเจอ node 1 ซึ่งเคย visit แล้ว ทำให้อัลกอริทึมนึกว่ามี cycle ทั้ง ๆ ที่มันไม่มี

นั่นคือ การที่จะเจอ cycle ได้นั้น ต้องเจอระหว่าง DFS ในรอบเดียวกันเท่านั้น ไม่ใช่เจอ node ที่เคย visited เพราะการ DFS รอบอื่น

```cpp
int color[N]; // เริ่มมาเป็น 0 หมด
bool dfs(int u) {
    if (color[u] == 1) // เจอ cycle
        return true;
    if (color[u] == 2) // เจอ node ที่เคยผ่านในรอบอื่นแล้ว
        return false;
    color[u] = 1; // ตอนเริ่มทำ เซตเป็นสีเทา
    for (auto v : G[u]) {
        if (dfs(v)) // ถ้า dfs แล้วเจอ cycle ก็ return true เลย
            return true;
    }
    color[u] = 2; // พอทำเสร็จแล้วก็เซตเป็นสีดำ
}

// main: คล้าย ๆ กับด้านบน
```

### 1.2.3 ใช้ DFS เพื่อนับจำนวน component ใน undirected graph

ทำคล้าย ๆ 1.2.1 แต่เราจะไม่ return อะไรทั้งสิ้น แค่ DFS เพื่อมาร์คว่าจุดไหนเคยไปแล้วเฉย ๆ ทำไปเรื่อย ๆ แล้วพบว่า เราต้องเริ่ม DFS ทั้งหมดกี่ครั้งจึงจะมาร์คครบทั้งกราฟ

```cpp
bool visited[N];
void dfs(int u) {
    if (visited[u])
        return;
    visited[u] = true;
    for (auto v : G[u])
        dfs(v);
}

// main:
int component = 0;
for (int u = 1; u <= n; ++u) {
    if (!visited[u]) {
        ++component;
        dfs(u); // dfs เพื่อมาร์คทุก node ใน component ว่าเคยเจอแล้ว
    }
}
```

นอกจากใช้นับจำนวน component แล้ว แทนที่จะใช้เป็น visited เฉย ๆ ก็อาจจะเก็บเลข component ไว้ด้วยก็ได้ (component แรกเป็นเลข 1, ถัดมาเป็นเลข 2 ...) ทำให้เราสามารถตอบได้อย่างรวดเร็วว่า node 2 node อยู่ component เดียวกันหรือไม่ (ถ้าตัวเลขเหมือนกันก็คือ component เดียวกัน)

### 1.2.4 ใช้ BFS หาเส้นทางสั้นสุดใน unweighted graph

เนื่องจากว่า Breadth-first Search จะมีลักษณะคล้าย ๆ การกระจายเป็นชั้น ๆ ออกไป ถ้าทุกเส้นเชื่อมในกราฟน้ำหนักเท่ากันหมด เราสามารถใช้ BFS ในการหาเส้นทางสั้นสุดจาก node นึงออกไปหาอีก node นึงได้

```cpp
int level[N];

void shortest_path(int start, int target) {
    // fill ทุกช่องเป็น -1 เปรียบเสมือน visited[u] = false
    fill(level, level+N, -1);

    queue<int> Q;
    level[start] = 0;
    Q.push(start);

    while (!Q.empty()) {
        int u = Q.front();
        Q.pop();
        if (u == target) // เจอคำตอบแล้ว
            return level[u];
        for (auto v : G[u]) {
            if (level[v] == -1) { // กระจายไปยัง node รอบ ๆ ที่ยังไม่เคยไป
                level[v] = level[u]+1;
                Q.push(v);
            }
        }
    }

    return -1; // ถ้าทำมาถึงตรงนี้แล้ว แปลว่าไม่มีเส้นทางจาก start ไปถึง target 
}
```

### 1.2.5 ใช้ DFS เพื่อเช็คความเป็น bipartite graph

กราฟจะเป็น Bipartite Graph ก็ต่อเมื่อเราสามารถระบายสี node แต่ละ node ด้วย 1 ใน 2 สี โดยที่ แต่ละ node ที่มีเส้นเชื่อมกันต้องมีสีต่างกัน

สามารถเช็คได้ง่าย ๆ โดยการ DFS ทดลองระบายสี โดยเราจะระบายสีสลับกันไปเรื่อย ๆ แต่ถ้าวนกลับมา node เดิมที่เคยระบายแล้วพบว่าสีที่ต้องการระบาย กับสีที่ระบายลงไปแล้วขัดแย้งกัน จะถือว่ากราฟดังกล่าวไม่เป็น Bipartite Graph

```cpp
int color[N]; // 0 = ยังไม่ได้ลงสี, 1 = ลงสีแรก, 2 = ลงสีที่สอง, ตอนแรกเป็น 0 หมด

// dfs, ต้องการระบายสี node u เป็นสี c
// return true ถ้ายัง*อาจจะ*เป็น bipartite graph อยู่, false ถ้าเป็นไม่ได้แน่นอน
void dfs(int u, int c) {
    if (color[u] != 0) { // ถ้าเคยระบายสีแล้ว
        // สีต้องตรงกัน ถ้าไม่ตรงกันจะต้อง return false
        return (color[u] == c);
    }
    color[u] = c; // ระบาย
    for (auto v : G[u]) {
        // ระบาย node รอบ ๆ แต่ว่ากลับสี
        if (!dfs(v, c == 1 ? 2 : 1)) // ถ้าเละ ก็ให้ return false
            return false;
    }
    return true; // ถ้ารอดมาได้ ก็ยังมีสิทธิ์เป็น bipartite graph
}

// main:
bool isbipartite = true;
for (int u = 1; u <= n; ++u) {
    if (color[u] == 0) {
        if (!dfs(u, 1)) { // ถ้าลองระบาย (เริ่มจากสี 1) ใน component นั้นแล้วเละ ก็ถือว่าไม่เป็น bipartite graph
            isbipartite = false;
            break;
        }
    }
}
// กราฟอาจจะ disconnect กัน, ต้องลองเติมสีให้ครบทุก component
```

## 1.3 Shortest Path Algorithm

หลัก ๆ แล้วเขียนแค่ Dijkstra’s Algorithm เป็นก็พอ ส่วน Bellman-Ford ควรรู้ไว้เผื่อโจทย์มี edge ที่มี negative weight

Floyd-Warshall ไม่จำเป็นก็ได้ แต่ถ้ามีโจทย์ที่ต้องใช้ All-Pair Shortest Path โค้ด Floyd-Warshall สะดวกมาก แค่ไม่กี่บรรทัดก็เสร็จแล้ว

นอกจากนี้ มี Shortest Path Faster Algorithm (SPFA) ซึ่งเป็นการดัดแปลงผสมผสาน Bellman-Ford กับ BFS เข้าด้วยกัน โค้ดได้ง่าย สามารถหาเส้นทางสั้นสุดได้รวดเร็ว (โดยเฉลี่ย จะหาได้เร็วกว่า Dijkstra's Algorithm) และรองรับกราฟที่มี negative weight ได้ด้วย

หากจำ Dijkstra's Algorithm ไม่ได้ แนะนำให้จำ SPFA ไว้ เผื่อจะช่วยได้ (ถึงอย่างไรก็ตาม ต้องระวังว่า Worst case อาจจะแย่พอ ๆ กับ Bellman-Ford Algorithm)

### 1.3.1 Dijkstra’s Algorithm

หลักการทำงานคร่าว ๆ
- ตอนแรกถือว่า distance ไปยังทุก node เป็น infinity ก่อน ยกเว้น node เริ่มต้นเป็น 0 นำใส่คิว
- พิจารณา node ที่ distance น้อยสุดในคิว แล้วปรับ distance ของ node รอบ ๆ (+ edge weight เข้าไป)
- ถ้าปรับได้ ก็ให้นำใส่คิว เพื่อพิจารณาปรับ node รอบ ๆ อีกเรื่อย ๆ จนกว่าจะจบ

ในที่นี้ assume ว่าเก็บกราฟแบบ Adjacency List ที่เป็น `vector<pair<int, int>> G[N]` โดยตัวแรกเก็บหมายเลข node ปลายทาง และตัวที่สองเก็บความยาวของ edge (ตามที่อธิบายด้านบน)

เราต้องการ Priority Queue เพื่อเก็บ node แล้ว sort ตามระยะทาง

วิธีที่ง่ายที่สุดคือ การเก็บ `pair<int, int>` ใน priority_queue โดยที่ตัวแรกเก็บระยะทาง ตัวที่สองเก็บ node เมื่อเก็บอย่างนี้ ตัว `priority_queue` จะ sort ตามตัวแรกก่อน ซึ่งก็คือการ sort ตามระยะทางตามที่เราต้องการนั่นเอง

เนื่องจาก `priority_queue` เป็น max-heap แต่เราต้องการ min-heap ดังนั้น เราต้องใช้ตัว compare เป็น `greater<pii>` แทน (เดิมเป็น `less<pii>`)

```cpp

using pii = pair<int, int>;

vector<int> dist(n+1, INF); // ที่ใช้ n+1 เพราะหมายเลข node ในกราฟอาจจะเป็น 1-based index
vector<bool> visited(n+1, false);

priority_queue<pii, vector<pii>, greater<pii>> Q;
dist[start] = 0;
Q.push({dist[start], start});

while (!Q.empty()) {

    int u = Q.top().second, d = Q.top().first;
    Q.pop();
    if (visited[u])
        continue;
    visited[u] = true;

    if (u == target) { // เจอคำตอบ
        printf("%d\n", dist[target]);
        break;
    }

    for (auto vw : G[u]) { // อย่าลืมว่า adjacency list เรามี (node ปลายทาง, weight)
        int v = vw.first;
        int w = vw.second;
        if (!visited[v] && dist[u]+w < dist[v]) { // ปรับ distance ให้น้อยลง
            dist[v] = dist[u]+w;
            Q.push({dist[v], v}) // ถ้าปรับได้ก็ยัดใส่ queue
        }
    }
}

```

Dijkstra's Algorithm ตามโค้ดนี้จะทำงานใน `O(m + n log n)` ซึ่งถือว่ารวดเร็วพอสมควร แต่ต้องระวัง ห้ามใช้ Dijkstra's Algorithm กับกราฟที่มี negative weight

### 1.3.2 Bellman-Ford Algorithm

Bellman-Ford Algorithm มีไว้เพื่อหาเส้นทางสั้นสุด เริ่มต้นจาก node หนึ่งไปยัง node อื่น ๆ ทุก node มีแนวคิดคร่าว ๆ ดังนี้
- นิยามให้ `dist[v]` แทนเส้นทางสั้นสุดจาก node เริ่มต้นไปยัง node `v`
- ตอนแรก ถือว่าทุก node มี `dist[v] = INF` ยกเว้น node เริ่มต้นจะมี `dist[v] = 0`
- พิจารณา edge `(u, v, w)` แต่ละ edge แล้วเปรียบเทียบเส้นทางที่มาถึง node `v` ได้สองเส้นทาง คือ 1) ระยะทางสั้นสุดเดิมที่มาถึง node `v` ได้ (`dist[v]`) และระยะทางสั้นสุดมาถึง node `u` ต่อด้วย edge ดังกล่าวมาถึง node `v` (`dist[u] + w`) นำค่าที่น้อยกว่าเก็บลงใน `dist[v]`
- เมื่อพิจารณาครบทุก edge แล้ว ให้พิจารณาซ้ำอีกรอบ ทำเรื่อย ๆ จนกว่าจะเจอรอบที่ไม่มีการเปลี่ยนแปลงค่า `dist[v]` ใด ๆ แล้วจึงหยุดการทำงานของโปรแกรม

สังเกตว่าวิธีนี้ จะต้องพิจารณา edge ซ้ำทั้งหมดไม่เกิน `n-1` รอบ เพราะ path จาก node เริ่มต้นไปยัง node ใด ๆ ที่เป็นเส้นทางสั้นสุด ย่อมผ่านเส้นไม่เกิน `n-1` เส้น ดังนั้น algorithm นี้จะทำงานในเวลา `O(nm)`

algorithm นี้สามารถใช้หาเส้นทางสั้นสุดได้ในกรณีที่กราฟมี edge ที่มี negative weight (ใช้ Dijkstra's Algorithm ไม่ได้)

ในกรณีที่กราฟมี negative cycle (cycle ที่มี weight รวมติดลบ) algorithm อาจจะไม่จบการทำงาน ดังนั้นให้นับจำนวนรอบที่พิจารณา edge ด้วย หากทำงานรอบที่ `n-1` แล้วยังพบว่ามีการเปลี่ยนแปลงค่า `dist[v]` ก็สามารถสรุปได้ว่ากราฟมี negative cycle

```cpp
// รับกราฟเป็น edge list
struct Edge {
    int u, v, w;
};
vector<Edge> edges;

// bellman-ford:

vector<int> dist(n+1, INF);
dist[start] = 0;

int count = 0;
bool found_changes = false;
bool neg_cycle = false;
do {
    found_changes = false;
    for (auto edge : edges) {
        int u = edge.u, v = edge.v, w = edge.w;
        if (dist[u]+w < dist[v]) {
            dist[v] = dist[u]+w;
            found_change = true;
        }
    }
    ++count;
    if (count > n-1 && found_changes) {
        neg_cycle = true;
        break;
    }
} while (found_changes);
```

### 1.3.3 Shortest Path Faster Algorithm

Bellman-Ford Algorithm ทำงานช้าเนื่องจากลำดับ edge ที่พิจารณาทำให้รอบ ๆ หนึ่งมีการปรับค่า `dist[v]` น้อย ยกตัวอย่างกราฟที่ประกอบไปด้วย `n = 5` nodes และ `m = 4` edges ได้แก่ `(1, 2, 1)`, `(2, 3, 1)`, `(3, 4, 1)`, `(4, 5, 1)` (เป็นกราฟเส้นตรงตั้งแต่ node 1 ถึง 5 แต่ละเส้นหนัก 1)

หาเส้นทางสั้นสุดจาก node 0 - เริ่มมา `dist[1] = 0` ส่วน `dist[2..5] = INF` สังเกตว่า ถ้าเราพิจารณา edge `(4, 5, 1)` ก่อน จะไม่มีการเปลี่ยนแปลง `dist[5]` เลย เพราะ `dist[5] >= dist[4]+1` เช่นเดียวกับ edge `(3, 4, 1)` และ `(2, 3, 1)` ทั้งนี้เป็นเพราะ `dist[2..5] = INF` อยู่ ไม่สามารถนำไปใช้ปรับ distance node อื่น ๆ ได้

สังเกตว่า edge เดียวที่ใช้ได้คือ edge `(1, 2, 1)` จะทำให้ `dist[2] = 1` ส่วน `dist[3..5] = INF` เหมือนเดิม หากเราเริ่มใช้ edge นี้ก่อนตั้งแตแรก ตามด้วย edge `(2, 3, 1)`, `(3, 4, 1)` และ `(4, 5 1)` ก็จะทำให้ปรับ distance เสร็จตั้งแต่รอบแรกทันที

ดังนั้น เราจะเลือก edge อย่างชาญฉลาด ดังนี้
- เก็บกราฟเป็น adjacency list แทน
- เก็บ queue ของ node ที่น่าสนใจ เริ่มมาสนใจเฉพาะ node เริ่มต้น
- เมื่อดึง node ออกจากคิว ให้พิจารณาทุก edge ที่ติดกับ node นั้นแล้วปรับ distance ของ node รอบ ๆ
- หากมีการเปลี่ยนแปลง ก็ให้ push node ที่มีการเปลี่ยนแปลงใส่คิว
- ทำไปเรื่อย ๆ จนกว่าจะไม่มีการเปลี่ยนแปลงใด ๆ หรือหากมี node ใดถูกปรับเกิน `n-1` ครั้ง ให้สรุปว่ามี negative cycle

algorithm นี้ โดยเฉลี่ยแล้วจะทำงานใน `O(m)` (ขอไม่กล่าวถึงการพิสูจน์ ณ ที่นี้) แต่มี worst case เป็น `O(nm)` เช่นเดียวกับ Bellman-Ford Algorithm แบบปกติ

สังเกตว่าลักษณะการโค้ดจะโค้ดคล้าย ๆ กับ Breadth-first Search - เพียงแค่ตัด visited array ออกไปเท่านั้น (อนุญาตให้มีการวนกลับมาปรับ node เดิมได้)

```cpp
// adjaceny list
using pii = pair<int, int>;
vector<pii> G[N];

// SPFA:

vector<int> dist(n+1, INF);
vector<int> count(n+1, 0);
dist[start] = 0;
queue<int> Q;
Q.push(start);

bool neg_cycle = false;
while (!Q.empty()) {
    int u = Q.front();
    Q.pop()
    if (++count[u] > n-1) {
        neg_cycle = true;
        break;
    }
    for (auto vw : G[u]) {
        int v = vw.first, w = vw.second;
        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;
            Q.push(v);
        }
    }
}
```

สังเกตว่า SPFA เป็น algorithm ที่เขียนง่ายมาก โจทย์ส่วนใหญ่ไม่ค่อยได้ generate test case ที่เป็น worst case ของ SPFA ไว้มากนัก ดังนั้น สามารถใช้ algorithm นี้แทน Dijkstra's Algorithm ได้ (ถึงอย่างไรก็ตาม ควรเขียน Dijkstra's Algorithm ดั้งเดิมเป็น)

### 1.3.4 Floyd-Warshall Algorithm

นิยามให้ `dist[u][v]` คือเส้นทางสั้นสุดจาก node `u` ไป node `v` ที่รู้ ณ ตอนนั้น
- ตอนแรก เราไม่รู้ระยะทางอะไรเลย ให้ทุกช่องเป็น INF ก่อน
- สำหรับทุก node `u` ให้เซต `dist[u][u] = 0` (เพราะทุก node สามารถเดินทางหาตัวเองได้เป็นระยะทาง 0)
- รับ edge `(u, v, w)` ให้เซต `dist[u][v] = w` (เพราะเรารู้ระยะทางแล้ว)

แล้วเราจะลูปพิจารณา node `k`, `i`, `j` ตามแนวคิดดังนี้
- พิจารณาเส้นทางสั้นสุดจาก `i` ไป `j`, เดิมมีค่าเท่ากับ `dist[i][j]` ตามตาราง
- ถ้าสมมุติ เราเดินทางอ้อมผ่าน node `k` คำตอบจะกลายเป็น `dist[i][k] + dist[k][j]`
- ถามว่า การเดินอ้อมช่วยให้คำตอบดีขึ้นมั้ย ถ้าช่วยให้คำตอบดีขึ้น ให้แก้ `dist[i][j] = dist[i][k]+dist[k][j]`

```cpp
int n, m;
scanf("%d%d", &n, &m);

// เซตค่า 0 กับ INF ตอนแรก
for (int i = 1; i <= n; ++i) {
    for (int j = 1; j <= n; ++j) {
        if (i == j)
            dist[i][j] = 0;
        else
            dist[i][j] = INF;
    }
}

// รับ edge ของกราฟมาทั้งหมด
for (int i = 0; i < m; ++i) {
    int u, v, w;
    scanf("%d%d%d", &u, &v, &w);
    dist[u][v] = w;
}

// floyd-warshall
for (int k = 1; k <= n; ++k) {
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= n; ++j)
            dist[i][j] = min(dist[i][j], dist[i][k]+dist[k][j]);
    }
}
```

สังเกตว่าโค้ดค่อนข้างสั้น อัลกอริทึมนี้ทำงานใน `O(n^3)` ซึ่งสามารถรองรับได้เพียงแค่กราฟขนาดเล็กเท่านั้น สามารถใช้อัลกอริทึมนี้กับกราฟที่มี negative weight ได้ การตรวจสอบ negative cycle ให้ดูค่า `dist[u][u]` ของทุก node `u` - หากมีค่าติดลบ จึงสรุปได้ว่ากราฟมี negative cycle

### 1.3.5 ปัญหาประยุกต์ Shortest Path

**Unweighted Graph** 

สำหรับกราฟที่ไม่ได้ระบุน้ำหนักไว้ (ทุกเส้นหนัก 1 เท่ากันหมด) แทนที่จะใช้ Dijkstra's Algorithm เราสามารถใช้ Breadth-first Search ในการหาเส้นทางสั้นสุดได้

**Multiple-Source Shortest Path**

ยกตัวอย่างโจทย์ ดังนี้
> มีตารางขนาด `n*m` ให้ บางช่องอาจมีกำแพง และบางช่องอาจมีเชื้อไวรัสอยู่ ทุก ๆ นาทีเชื้อไวรัสจะแพร่ไปยังช่องบน/ล่าง/ซ้าย/ขวา (แต่จะไม่แพร่ผ่านกำแพง) ให้หาว่าแต่ละช่องจะถูกไวรัสแพร่มานาทีที่เท่าใด (หากมีไวรัสตั้งแต่แรก ให้ถือว่าแพร่มานาทีที่ 0)
>
> เช่น หากตารางขนาด `4*5` มีลักษณะดังนี้ (ช่องที่มี `o` คือมีเชื้อไวรัสอยู่ ส่วน '#' คือกำแพง)
> ```
> - - - # o
> - - - # -
> - # - - -
> o - - - -
> ```
> เชื้อไวรัสจะแพร่กระจายไปยังช่องต่าง ๆ ตามเวลาที่ระบุดังนี้
> ```
> 3 4 5 # 0
> 2 3 4 # 1
> 1 # 3 3 2
> 0 1 2 3 3
> ```

สังเกตว่า แต่ละช่อง ไวรัสที่อยู่ใกล้ที่สุดจะแพร่เข้ามาหา ดังนั้น คำตอบของแต่ละช่องจะเท่ากับ เส้นทางสั้นสุดจากตำแหน่งช่องนั้นไปหาแหล่งไวรัสที่ใกล้ที่สุด วิธีนี้จะใช้เวลานานมาก เพราะเราต้องเริ่มหา shortest path จากทุกจุดเริ่มต้น เนื่องจากมีจุดเริ่มต้นมากสุดถึง `nm` จุด และเราต้อง BFS ทั่วทั้งตารางทั้งหมด `nm` ช่อง รวมแล้ว Time Complexity จะเป็น `O((nm)^2)`

หากเราเปลี่ยนมาพิจารณาแหล่งกำเนิดไวรัสแต่ละแหล่งแทน เราสามารถหาได้ว่า ไวรัสตัวนั้นจะแพร่ไปยังช่องใด ณ เวลาใด ยกตัวอย่างตามกรณีข้างบน ไวรัสตัวมุมบนขวาจะแพร่ในลักษณะนี้
```
8 7 6 # 0
7 6 5 # 1
8 # 4 3 2
7 6 5 4 3
```
ส่วนไวรัสมุมล่างซ้ายจะแพร่ในลักษณะนี้
```
3 4 5 # 7
2 3 4 # 6
1 # 3 4 5
0 1 2 3 4
```
หากเรานำทั้งสองตารางมารวมกัน โดยสนใจเฉพาะค่าที่น้อยที่สุดของแต่ละช่อง ก็จะได้คำตอบตามที่อธิบายไว้ทางด้านบน

วิธีนี้ เราลดจำนวนจุดเริ่มต้นลงเหลือเท่ากับจำนวนแหล่งกำเนิดไวรัส หากมีแหล่งกำเนิดไวรัส `k` จุด จะใช้เวลาทำงานทั้งหมด `O(knm)` ถึงอย่างไรก็ตาม แหล่งกำเนิดไวรัสอาจมีได้มากถึง `O(nm)` จุด ทำให้วิธีนี้ไม่ได้ดีกว่าเดิมมากนัก

วิธีที่ดีที่สุด คือการทำ BFS พร้อมกัน แทนที่จะมีจุดเริ่มต้นเพียงจุดเดียว เราจะเริ่มต้นหลาย ๆ จุด เมื่อทำเช่นนี้ เราจะได้เวลาของตารางแต่ละช่องทันที วิธีนี้ใช้เวลา `O(nm)` ซึ่งดีที่สุดเท่าที่เป็นไปได้แล้ว

**Shortest Path ผ่าน node ที่กำหนด**

> กำหนดกราฟ**ไม่มีทิศทาง**มาให้ ให้หาเส้นทางสั้นสุดจาก node `u` ไป node `v` แต่มีเงื่อนไขพิเศษคือ เส้นทางต้องผ่าน node `x` - ให้ตอบเส้นทางสั้นสุด เมื่อพิจารณา node `x = 1, 2, 3, ..., n`

สังเกตว่า ความยาวทั้งหมดจะเท่ากับ `dist[u][x] + dist[x][v]` เมื่อกำหนดให้ `dist[a][b]` คือเส้นทางสั้นสุดจาก `a` ไป `b`

ถึงอย่างไรก็ตาม เมื่อเราเปลี่ยน node `x` สังเกตว่าเราต้องทำ shortest path เริ่มต้นจาก node `x` ใหม่เสมอ นั่นแปลว่าเราจะต้องทำ shortest path ซ้ำถึง `n` ครั้ง (หรือไม่ก็ต้องทำ Floyd-Warshall ซึ่งช้าพอ ๆ กัน)

สังเกตว่า เราสามารถแก้สมการเป็น `dist[u][x] + dist[v][x]` ได้ เนื่องจากจุดเริ่มต้น `u` และ `v` ไม่มีการเปลี่ยนแปลง นั่นแปลว่าเราต้องทำ shortest path เพียง 2 ครั้งเท่านั้น แล้วลูปทดลองค่า `x` เพื่อหาคำตอบที่น้อยที่สุดได้

อนึ่ง หากเป็นกราฟแบบมีทิศทาง `dist[x][v]` อาจไม่เท่ากับ `dist[v][x]` ทำให้ใช้วิธีดังกล่าวโดยตรงไม่ได้

ถึงอย่างไรก็ตาม `เส้นทางสั้นสุดจาก x ไป v ในกราฟปกติ` จะเท่ากับ `เส้นทางสั้นสุดจาก v ไป x ในกราฟกลับทิศทาง` เราสามารถใช้ประโยชน์จากความจริงนี้ได้ โดยการกลับทิศทาง edge ทุกเส้น แล้วหาเส้นทางสั้นสุดออกจาก node `v` โดยให้กำหนด `เส้นทางสั้นสุดจาก v ไป x ในกราฟกลับทิศทาง` เป็น `dist[x][v]` ของกราฟเดิม เราจะใช้วิธีลูป node `x` เพื่อหาค่า `dist[u][x] + dist[x][v]` ที่น้อยที่สุดได้

**Shortest Path on States**

> กำหนดกราฟให้ ให้หาเส้นทางสั้นสุดจาก `u` ไป `v` โดยมีเงื่อนไขคือเส้นทางจะต้องผ่านจำนวน edge เป็นพหุคูณของ `k` (`k` มีค่าน้อย) เช่น หาก `k=2` เส้นทางจะต้องผ่าน 2, 4, 6, 8, 10, ... edges

สำหรับโจทย์ข้อนี้ เราจำเป็นต้องนิยามกราฟใหม่ขึ้นมา แล้วหา shortest path บนกราฟใหม่ เพราะมีเงื่อนไขจำกัดกว่าเดิม

กำหนดให้กราฟเดิมเป็นกราฟ `G` ส่วนกราฟใหม่เป็นกราฟ `G'`
- สำหรับ node `u` ในกราฟ `G` เราจะถือว่ามี node `(u, 0)`, `(u, 1)`, `(u, 2)`, ..., `(u, k-1)` ในกราฟ `G'`
- สำหรับ edge `(u, v, w)` ในกราฟ `G` เราจะถือว่ามี edge `((u, x), (v, (x+1)%k), w)` สำหรับ `x = 0..k-1` ในกราฟ `G'`

สังเกตว่ากราฟใหม่ที่เราสร้างขึ้นมา จะคล้ายคลึงกับกราฟเก่า แต่จะแบ่งเป็นชั้น ๆ ว่าเราเดินผ่าน edge มาเป็นจำนวนกี่เส้น ทุกครั้งที่เราเดินผ่าน edge ในกราฟเดิม จำนวน edge ที่จดไว้จะเพิ่มขึ้น ถ้าครบ `k` ก็จะวนกลับไปนับ 0 ใหม่

เราต้องหาเส้นทางสั้นสุดจาก node `(u, 0)` ไปยัง node `(v, 0)` ในกราฟนี้ จึงจะได้คำตอบของปัญหาเดิม ซึ่งก็คือ เส้นทางสั้นสุดจาก node `u` ไป node `v` ที่ผ่านจำนวน edge เป็นพหุคูณของ `k` พอดี (สังเกตว่าถ้าเป็นพหุคูณของ `k` จำนวน edge ที่ผ่าน จะต้องวนกลับมาที่ 0 พอดี)

อนึ่ง เนื่องจากเราหาเส้นทางสั้นสุดบนกราฟใหม่ การเก็บตาราง `dist` หรือ `visited` จะต้องใช้เป็น array 2 มิติแทน เช่น node `(u, x)` ต้องเก็บไว้ที่ช่อง `dist[u][x]` เป็นต้น

> กำหนดตารางให้ แต่ละช่องอาจจะเป็นกำแพง ประตู หรือกุญแจ ต้องการเดินจากจุดหนึ่งไปยังอีกจุดหนึ่ง โดยที่เราจะเดินผ่านประตูได้ก็ต่อเมื่อเคยเดินผ่านกุญแจของประตูนั้นแล้ว (คู่ประตู-กุญแจมีไม่กี่คู่)

สำหรับโจทย์ข้อนี้ สังเกตว่า เราสามารถแยกกราฟเป็นชั้น ๆ ได้เช่นกัน แทนที่จะเป็นแค่ตำแหน่ง `(x, y)` ก็อาจจะเก็บไว้ด้วยว่า เคยเก็บกุญแจไหนแล้วบ้าง เช่น หากมีคู่ประตู-กุญแจทั้งหมด 3 คู่ ก็อาจจะเก็บเป็น `(x, y, 0, 1, 1)` ถ้าเคยเก็บกุญแจที่ 2 กับ 3 แล้ว

สังเกตว่าค่าด้านหลัง เราสามารถใช้ตัวเลขปกติเก็บได้ แล้วใช้ bitwise operation ในการตรวจสอบว่าเคยเก็บกุญแจนั้นแล้วหรือยัง หรือใช้ในการกำหนดว่าเคยเก็บกุญแจแล้ว

เราทำ BFS ตามปกติ แต่เมื่อเดินไปในตำแหน่งที่มีกุญแจ แทนที่จะนำ `(x, y, list)` ใส่คิว เราต้องปรับ list กุญแจก่อน แล้วจึงนำใส่คิวได้

เช่นเดียวกับข้อก่อนหน้า ตาราง `dist` และ `visited` ต้องใช้ตาม node ก็คือ `(x, y, list)` ต้องเก็บไว้ใน `dist[x][y][list]`

## 1.4 Minimum Spanning Tree

Minimum Spanning Tree (MST) คือต้นไม้แผ่ทั่วของกราฟ ที่มีผลรวมของน้ำหนักของเส้นเชื่อมน้อยที่สุดเท่าที่เป็นไปได้ หลัก ๆ แล้วมี algorithm ที่ใช้แก้ 2 อย่างคือ Kruskal's Algorithm และ Prim's Algorithm

### 1.4.1 Kruskal's Algorithm

หลักการคร่าว ๆ ดังนี้
- ตอนแรก เราจะมีกราฟว่าง ๆ ที่มีแต่ node แต่ไม่มี edge
- sort edge เรียงตามน้ำหนักจากน้อยไปมาก
- ค่อย ๆ ทดลองเพิ่มทีละ edge โดยจะเพิ่มได้ก็ต่อเมื่อเพิ่มแล้วทำให้ไม่เกิด cycle
- ถ้าเพิ่มได้ก็เพิ่มเลย แต่ถ้าเพิ่มไม่ได้ก็ข้าม edge นั้นไป
- ทำไปเรื่อย ๆ จนกว่าครบทุก edge

ปัญหาจะอยู่ที่การเช็คว่าการเพิ่ม edge `(u, v)` จะทำให้เกิด cycle หรือไม่ วิธีการเช็คหลัก ๆ แล้วมีสามวิธี
- เช็คว่า node `u` และ `v` อยู่ component เดียวกันหรือไม่โดยการ DFS - ถ้าพบว่าอยู่ component เดียวกัน เราไม่ควรเพิ่ม edge `(u, v)` เพราะจะทำให้เกิด cycle
- ทดลองเพิ่มไปเลย แล้วใช้ DFS เช็ค cycle ในกราฟ ถ้าพบว่ามี cycle ก็ค่อยลบ edge `(u, v)` ทิ้ง
- ใช้ Data Structure ที่ใช้ว่า Union-find Disjoint set (DSU) เพื่อที่จะเช็คได้อย่างรวดเร็วว่าอยู่ใน component เดียวกันหรือไม่

โดยปกติแล้ว เวลา implement Kruskal's เราจะใช้ DSU เพราะวิธีอื่นจะเสียเวลาในการทำ Graph Traversal เป็นอย่างมาก

สำหรับรายละเอียดของ DSU ให้อ่านในหัวข้ออื่น ๆ ตอนนี้ทราบแค่ว่า DSU ทำได้สองอย่างคือ

1. `merge(u, v)` การรวม component ของ node `u` กับ `v` เข้าด้วยกัน (เพิ่ม edge `(u, v)`)
2. `find(u)` หาว่า node `u` อยู่ component ใด (เราเทียบ node `u` กับ node `v` - ถ้าอยู่ component เดียวกัน ห้ามเพิ่ม edge)

โค้ดของ Kruskal's Algorithm เป็นดังนี้

```cpp
// disjoint set functions
int find(int u);
int merge(int u, int v);

// สมมุติว่าเก็บกราฟเป็น edge list
struct Edge {
    int u, v, w;
    bool operator<(const Edge &rhs) const {
        return w < rhs.w;
        // ฟังก์ชันสำหรับ compare Edge (compare ตาม weight)
    }
};
vector<Edge> edges;

// kruskal's algorithm
sort(edges.begin(), edges.end());
int sum = 0;
for (auto edge : edges) { // พิจารณาทีละ edge เรียงจากน้อยไปมาก
    if (find(edge.u) == find(edge.v))
        continue; // ถ้าอยู่ component เดียวกัน ห้ามเพิ่ม edge
    // ถ้าไม่ได้อยู่ component เดียวกัน เพิ่ม edge ได้
    sum += edge.w;
    merge(edge.u, edge.v);
}
printf("%d\n", sum);
```

### 1.4.2 Prim's Algorithm

หลักการทำงานคร่าว ๆ ดังนี้
- เราจะเริ่มต้นจาก node ใดก็ได้ node หนึ่งก่อน
- เลือก node ที่อยู่ชิดกับ node นี้ ที่เชื่อมโดย edge ที่สั้นที่สุด แล้วแผ่ไปยัง node นั้น
- ตอนนี้มี 2 node ละ ใน 2 node นี้ก็ให้พิจารณา node รอบ ๆ แล้วเลือก node ที่เราแผ่ไปได้สั้นที่สุด เป็น 3 node
- ทำเรื่อย ๆ จนกว่าจะแผ่ครบทั้งกราฟ

(รู้สึกว่าอธิบายยาก แนะนำให้ดู https://visualgo.net/en/mst ดีกว่า)

ในการโค้ด เราจะใช้ `dist` array โดยนิยามให้ `dist[u]` คือ edge ที่สั้นที่สุดที่สามารถเชื่อมจาก node ที่ถูกแผ่นมาถึงแล้ว มาถึง node `u` แล้วใช้ priority queue เพื่อเลือกแผ่ไปยัง node `u` ที่มีค่า `dist[u]` น้อยที่สุด

สังเกตว่าลักษณะคล้ายกับ Dijkstra's Algorithm มาก แค่เปลี่ยนนิยาม `dist[u]` จากเส้นทางสั้นสุดเป็น edge สุดท้ายที่สั้นที่สุดเฉย ๆ

จุดที่แตกต่างในโค้ด
- จากเดิม สูตรปรับ `dist[v] = dist[u]+w` เราใช้เป็น `dist[v] = w` เฉย ๆ (การเปรียบเทียบก็เช่นกัน)
- เราจะเก็บ sum ของ dist node ที่เลือกไว้ด้านนอกตลอด

```cpp

using pii = pair<int, int>;

vector<int> dist(n+1, INF);
vector<bool> visited(n+1, false);

priority_queue<pii, vector<pii>, greater<pii>> Q;
dist[start] = 0;
Q.push({dist[start], start});

int sum = 0;
while (!Q.empty()) {

    int u = Q.top().second, d = Q.top().first;
    Q.pop();
    if (visited[u])
        continue;
    visited[u] = true;
    sum += dist[u]; // <-- เพิ่ม edge เข้า MST

    for (auto vw : G[u]) {
        int v = vw.first;
        int w = vw.second;
        if (!visited[v] && w < dist[v]) { // ปรับ edge weight ให้น้อยลง
            dist[v] = w;
            Q.push({dist[v], v}) // ถ้าปรับได้ก็ยัดใส่ queue
        }
    }
}

```

เช่นเดียวกับ Dijkstra's Algorithm - Prim's Algorithm จะทำงานใน `O(m + n log n)`

สำหรับ Complete Graph แนะนำให้ใช้ Prim's Algorithm แบบดัดแปลง แทนที่จะใช้ Priority Queue เราจะใช้การลูปหา node ที่ `dist` น้อยสุดแทน จะทำให้ time complexity เป็น `O(n^2)`

### 1.4.3 ปัญหาประยุกต์ MST

โจทย์บางข้อ ไม่ได้ถามหา MST ตรง ๆ แต่ต้องประยุกต์ใช้ MST จึงจะหาคำตอบได้ ยกตัวอย่างเช่นโจทย์ข้อนี้

> กำหนดกราฟไม่มีทิศทาง/ไม่มีน้ำหนักให้ ให้หาว่า หากต้องการเดินทางจาก node `u` ไปยัง node `v` จะต้องเสียพลังงานน้อยสุดเท่าใด โดยพลังงานจะพิจารณาจาก **น้ำหนักของ edge ที่หนักที่สุดที่ต้องเดินผ่าน** (ส่วน edge อื่น ๆ ไม่สน)

สังเกตว่านี่ไม่ใช่โจทย์ shortest path เพราะเราสนใจเพียงแค่ edge ที่หนักที่สุดเท่านั้น โจทย์ข้อนี้สามารถแก้ได้โดยการดัดแปลง Kruskal's Algorithm ดังนี้

- เราจะ sort edge จากน้อยไปมาก แล้วค่อย ๆ เพิ่มทีละ edge เช่นเดิม
- ให้หยุด algorithm ทันทีที่มีเส้นทางจาก `u` ไป `v` แล้วตอบ edge weight สุดท้ายที่เพิ่ม (ซึ่งเป็น edge ที่หนักที่สุด)

สังเกตว่า ทันทีที่มีเส้นทางจาก `u` ไป `v` นั่นคือ เราพบคำตอบที่ดีที่สุดแล้ว เพราะตลอด algorithm เราพิจารณา edge ที่น้อยที่สุดที่เป็นไปได้ตลอด

นอกจากนี้ อัลกอริทึมสำหรับหา Minimum Spanning Tree สามารถนำมาใช้หา **Maximum** Spanning Tree ได้ เพียงแค่ปรับการ sort เรียงจากมากไปน้อยเท่านั้น

## 1.5 Topological Sorting

สำหรับกราฟระบุทิศทางที่ไม่มี cycle (Directed Acyclic Graph หรือ DAG) เราสามารถหมายเลข node เป็นลำดับได้ โดยที่ สำหรับทุก edge `(u, v)` node `u` จะอยู่ก่อน `v` ในลำดับเสมอ (กราฟหนึ่งอาจจะสร้าง Topological Sorting ได้หลายลำดับ)

เพื่อให้เข้าใจง่ายขึ้น เราอาจจะมองว่ากราฟเป็นตัวระบุลำดับการทำงานว่า เราต้องทำงานอะไรก่อนหลัง (edge `(u, v)` คือต้องทำงานชิ้นที่ `u` ก่อนจึงอนุญาตให้ทำงานชิ้นที่ `v`) ยกตัวอย่าง หากกราฟมี 4 node และมี edge `(1, 4)`, `(3, 2)`, `(3, 1)` นั่นหมายความว่า
- เราต้องทำงาน `1` ก่อนงาน `4`
- เราต้องทำงาน `3` ก่อนงาน `2`
- เราต้องทำงาน `3` ก่อนงาน `1`

ลำดับการทำงานที่เป็นไปได้ เช่น `3, 1, 2, 4` สังเกตว่าลำดับนี้ไม่ขัดกับเงื่อนไขที่กำหนดให้ ดังนั้น ลำดับนี้ถือว่าเป็น Topological Sorting (อาจจะย่อว่า Topo-sort) ของกราฟดังกล่าว

สังเกตว่า ถ้ากราฟมี cycle เราจะไม่สามารถหาลำดับได้ เช่น หากกราฟมี edge `(1, 2)` และ edge `(2, 1)` เราต้องทำงาน `1` ก่อนงาน `2` แต่เราต้องทำงาน `2` ก่อนงาน `1` เช่นกัน ซึ่งขัดแย้งกัน

การหา Topological Sorting สามารถทำได้ดังนี้
- หา in-degree ของแต่ละ node, in-degree แสดงถึงจำนวนงานที่ต้องทำก่อนที่จะทำงานดังกล่าวได้ (node ที่มี in-degree = 0 ก็คือ เราสามารถทำงานนั้นได้ตั้งแต่แรก)
- นำ node ที่มี in-degree = 0 ใส่คิวพิจารณา
- นำ node นึงในคิวมาใส่ลำดับ topo-sort แล้วลด in-degree ของ node รอบ ๆ ไป 1 (เปรียบเสมือนว่าทำงานนั้นเสร็จแล้ว)
- ถ้าลด in-degree ของ node ใดแล้วกลายเป็น 0 นั่นแปลว่างานนั้นเราสามารถทำได้แล้ว ก็ให้นำ node นั้นใส่คิวพิจารณาด้วย
- ทำไปเรื่อย ๆ จนกว่าจะนำทุก node ใส่ในลำดับ topo-sort ครบ

ถ้าเรานำ node ใส่ sequence ได้ไม่ครบทุก node แปลว่ากราฟดังกล่าวมี cycle ซึ่งทำให้เราไม่สามารถหา topological sorting ได้

```cpp
int indeg[N];
// สำหรับทุก edge (u, v) ให้ ++indeg[v]

queue<int> Q;
for (int u = 1; u <= n; ++u) {
    if (indeg[u] == 0)
        Q.push(u);
}

vector<int> seq; // sequence
while (!Q.empty()) {
    int u = Q.front();
    Q.pop();
    seq.push_back(u); // นำใส่ topo-sort
    for (auto v : G[u]) {
        --indeg[v]; // ลด in-degree node อื่น ๆ
        if (indeg[v] == 0) // ถ้าเหลือ 0 ก็ใส่คิว
            Q.push(v);
    }
}

// print sequence
// if seq.size() != n: fail
```

อัลกอริทึมนี้เรียกว่า Kahn's Algorithm โดยทำงานได้ใน `O(n+m)` 

นอกจากนี้ เราสามารถใช้ DFS ในการหา topo-sort ได้ สามารถศึกษาได้ที่ https://www.geeksforgeeks.org/topological-sorting/

## 1.6 Bridges and Articulation Point

กำหนดกราฟที่เชื่อมต่อกัน (Connected Graph) มาให้ เราจะเรียก node `u` ว่าเป็น Articulation Point (หรือ Cut Vertex) ก็ต่อเมื่อ การตัด node `u` ออก จะทำให้กราฟขาดออกจากกัน (ไม่ connected)

เราจะเรียก edge `(u, v)` ว่าเป็น Bridge ก็ต่อเมื่อ การตัด edge `(u, v)` ออก จะทำให้กราฟขาดออกจากกัน (ไม่ connected)

หากโจทย์ต้องการให้หา Bridge หรือ Articulation Point วิธีที่ง่ายที่สุดวิธีหนึ่งคือการทดลองตัดทุก node/edge แล้วทำการ DFS เพื่อหาว่ากราฟยังคงมี component เดียวอยู๋หรือไม่

วิธีนี้ หากใช้หา Articulation Point จะใช้เวลา `O(V(V+E))` หากใช้หา Bridge จะใช้เวลา `O(E(V+E))` ซึ่งใช้ไม่ได้ หากกราฟมีขนาดใหญ่

เราสามารถใช้ Tarjan's Algorithm หา Articulation Point หรือ Bridge ได้ใน `O(V+E)` ดังนี้

### 1.6.1 นิยามเกี่ยวกับ edge ใน Depth-first Search

ในการทำ DFS สังเกตว่า เมื่ออยู่ที่ node `u` แล้วเราพิจารณา node `v` ซึ่งอยู่ติดกับ node `u` และไม่ใช่ parent ในการ DFS ของ node `u` จะมีอยู่สองความเป็นไปได้
1) ยังไม่เคย visit node `v` (ต้อง recursive ต่อไป)
2) เคย visit node `v` อยู่แล้ว (นั่นคือ เจอ cycle, return)

เราจะเรียก edge `(u, v)` ว่าเป็น tree edge หากตรงตามเงื่อนไขที่ 1 แต่หากตรงตามเงื่อนไขที่ 2 เราจะเรียกว่าเป็น back edge

หากวาดกราฟโดยใช้เฉพาะ tree edge สังเกตว่าเราจะได้ spanning tree ออกมา ส่วน back edge เป็นเหมือนส่วนเสริมที่ทำให้เกิด cycle ต่าง ๆ ในกราฟ

### 1.6.2 นิยาม Discovery Time และ Lowlink Value

ระหว่างทำ DFS เราจะเก็บตัวแปรเพื่อนับลำดับ node ที่เจอ โดยทุกครั้งที่เจอ node ใหม่ เราจะเพิ่มค่า counter ดังกล่าว แล้วเก็บค่าไว้ใน `disc[u]` (`u` คือ node ใหม่ที่พิจารณาอยู่)

ต่อมา ขอนิยามให้ `low[v]` หมายถึง ค่า `disc` ที่น้อยที่สุดที่เป็นไปได้ หากเดินทางลงไปใน subtree บน DFS tree ของ node `v` แล้วเดินทางผ่าน back edge ไม่เกิน 1 เส้น

เมื่อพิจารณา node `u` อยู่ ตอนแรกเราจะกำหนดให้ `low[u] = disc[u]` แล้วเมื่อพิจารณา node รอบ ๆ node `u` - สมมุติว่าพิจารณา node `v` อยู่ (`v` ไม่ใช่ parent ของ node `u` ในการ DFS)

ถ้า `(u, v)` เป็น tree edge (นั่นคือยังไม่เคย visit node `v`) ให้ทำการ DFS ไปยัง node `v` จนเสร็จสิ้น แล้วกำหนดให้ `low[u] = min(low[u], low[v])` (เพราะ `low[v]` จะเก็บค่าน้อยสุดที่เป็นไปได้ หากเราเดินทางผ่าน `(u, v)` ลงไป ก็จะเดินทางไปถึงค่าดังกล่าวได้เช่นกัน นำมากำหนดเป็นค่า `low[u]` ได้)

ถ้า `(u, v)` เป็น back edge (นั่นคือ เคย visit node `v` แล้ว) ไม่จำเป็นต้อง DFS ซ้ำแล้ว แต่ให้กำหนด `low[u] = min(low[u], disc[v])` - สังเกตว่าในทีนี้เราไม่สามารถใช้ `low[v]` ในการปรับค่าได้แล้ว เพราะเราะเดินทางผ่าน `(u, v)` ซึ่งเป็น back edge ไปแล้วเส้นหนึ่ง ตามนิยาม จะใช้ back edge ต่ออีกเส้นหนึ่งไม่ได้

### 1.6.3 คุณสมบัติของ Articulation Point

สำหรับ node `u` ใด ๆ หากพิจารณา tree edge `(u, v)` แล้วพบว่า `low[v] >= disc[u]` นั่นคือ การเดินทางลงไป node `v` เราไม่สามารถหา back edge กลับขึ้นมาเหนือ node `u` ได้ จะสรุปได้ว่า node `u` เป็น articulation point

หากตัด node `u` ทิ้ง จะทำให้กราฟขาดออกจากกันแน่นอน เพราะ node ตั้งแต่ `v` ลงไป ไม่มีทางกลับขึ้นมาเหนือ node `u` ได้ นั่นคือ ไม่สามารถติดต่อกับส่วนอื่น ๆ ของกราฟได้

อนึ่ง มีข้อยกเว้น กรณีที่ node `u` เป็น root ของการ DFS จะใช้เงื่อนไขดังกล่าวไม่ได้ เพราะยังไงก็ไม่มี node ไหนเชื่อมขึ้นไปเหนือ root ได้อีกแล้ว เราจะใช้เงื่อนไขจำนวน node ลูกแทน หากมีจำนวน node ลูกซึ่งเชื่อมต่อกันด้วย tree edge มากกว่า 1 node จะถือว่า node `u` เป็น articulation point

### 1.6.4 คุณสมบัติของ Bridge

สำหรับ tree edge `(u, v)` ใด ๆ หากพบว่า `low[v] > disc[u]` นั่นแปลว่า ทุก node ตั้งแต่ `v` ลงไป ไม่มี back edge กลับขึ้นมาเหนือเส้น `(u, v)` ได้ จะสรุปได้ว่าเส้น `(u, v)` เป็น Bridge

### 1.6.5 โค้ดของ Tarjan's Algorithm

```cpp
// adjacency list
vector<int> G[N];

bool visited[N];
int disc[N], low[N];

set<int> ap; // answer: articulation points
set<pii> bridge; // answer: bridges

int counter = 0;
void tarjan(int u, int p) { // p = parent of u
    visited[u] = true;
    low[u] = disc[u] = ++counter;
    int child = 0;
    for (auto v : G[u]) {
        if (!visited[v]) {
            ++child;
            tarjan(v, u);
            low[u] = min(low[u], low[v]);
            // articulation point
            // ตามเงื่อนไขด้านบน (ในที่นี้ ให้ root มี parent เป็น 0)
            if ((p != 0 && low[v] >= disc[u]) || (p == 0 && child > 1))
                ap.insert(u);
            // bridge
            if (low[v] > disc[u])
                bridge.insert(pii(u, v));
        } else if (v != p) {
            low[u] = min(low[u], disc[v]);
        }
    }
}
```

อนึ่ง เราไม่จำเป็นต้องใช้ `visited` array ก็ได้ เพราะตอนแรก `disc` ทุกช่องทีย่ังไม่เคย visit จะมีค่าเท่ากับ 0 - ดังนั้นเช็คค่าจาก `dist` เอาก็ได้

