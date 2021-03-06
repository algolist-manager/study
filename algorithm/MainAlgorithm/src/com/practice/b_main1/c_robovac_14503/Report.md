# [연습] 로또 6603

* 시뮬레이션

## 문제 설명

<pre>
입력 
    - 첫째 줄 : 세로 크기 N & 가로 크기 M (3 <= N, M <= 50)
    - 둘째 줄 : 로봇 청소기 현 좌표 (r, c) & 현재 바라보는 방향 d 
              d == 0 (북) / d == 1 (동) / d == 2 (남) / d == 3 (서)
    - 셋째 줄부터 N 개의 줄 : 장소의 상태 주어짐
              0 빈칸 / 1 벽 
              장소의 모든 외곽은 벽
              로봇 청소기 현 좌표의 상태는 0
    ex )
    3 3
    1 1 0
    1 1 1
    1 0 1
    1 1 1
             
출력
    - 로봇 청소기가 청소하는 칸의 개수 출력
</pre>

로봇 청소기 작동 방식
1. 현재 위치 청소
2. 현재 위치에서 현재 방향 기준으로 '왼쪽 방향'부터 차례대로 탐색
    * 아직 청소 안 한 장소 : 그 방향으로 회전 후 전진 -> 1번으로 돌아감
    * 청소할 공간 없음 : 그 방향으로 회전 -> 2번으로 돌아감
    * 4방향 모두 청소되어 있거나 벽인 경우 : 방향 유지 -> 한 칸 후진 -> 2번으로 돌아감
    * 4방향 모두 청소되어 있거나 벽이면서, 뒤에도 벽이라 후진 불가능한 경우 : 작동 멈춤

## 문제 풀이

### 1. 나머지 계산 활용
* 방향 회전 
    * 0(북) 1(동) 2(남) 3(서) - 왼쪽 방향 회전 -> 3(서) 0(북) 1(동) 2(남)
    * (x + 3) % 4 의 결과값으로 변한다.
    ~~~
    int nd = (d + 3) % 4;
    ~~~

* 후진
    * 0(북) 1(동) 2(남) 3(서) - 후진 -> 2(남) 3(서) 0(북) 1(동)
    * (x + 2) % 4 의 결과값으로 변한다.
    ~~~
    int nd = (d + 2) % 4;
    ~~~

### 2. 장소의 상태
~~~
int[][] map = new int[n][m];
~~~
이중배열 map 에 장소의 상태에 대한 정보를 저장한다
0 : 아직 청소하지 않은 곳
1 : 벽
2 : 청소를 한 곳

로봇 청소기가 작동을 멈췄을 때, 배열 map 에서 2의 개수를 찾으면 답이다.
~~~
int ans = 0;
for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
        if (map[i][j] == 2) {
            ans += 1;
        }
    }
}
System.out.println(ans);
~~~

### 3. 문제를 나눈다.
<pre>
1. 현재 위치에서 최대 4 방향으로 돌릴 가능성이 있다.

2. 다음으로 갈 곳이 벽(1) 이거나 이미 청소한 장소(2)일 때의 로직

3. 다음으로 갈 곳이 아직 청소하지 않은 장소(0)일 때의 로직 
</pre>

* 4방향으로 돌리기
~~~
static int[] dx = {-1, 0, 1, 0};
static int[] dy = {0, 1, 0, -1};
~~~

~~~
Pair p = q.remove();
int x = p.x;
int y = p.y;
int d = p.d;

int nx;
int ny;
int nd;

for (int i = 0; i < 4; i++) {
    nd = (d + 3) % 4;
    nx = x + dx[nd];
    ny = y + dy[nd];
    
    ...
}
~~~


nd : 왼쪽 방향으로 회전한 방향  
nx : 회전된 방향으로 전진했을 때의 x 좌표  
ny : 회전된 방향으로 전진했을 때의 y 좌표

* 다음으로 갈 곳이 벽(1) 이거나 이미 청소한 장소(2)일 때의 로직
    1. 3번까지는 계속 방향을 돌려본다.
    2. 4번째는 후진
        * 후진했는데, 이미 청소한 장소 -> 후진 전 방향 유지하면서 후진
        * 후진하려고 했지만, 벽 -> 작동을 멈춘다.

~~~
boolean isCompleted = false;

while(!isCompleted) {

    Pair p = q.remove();
    int x = p.x;
    int y = p.y;
    int d = p.d;
    
    int nx;
    int ny;
    int nd;

    // Todo : 4 방향으로 돌리기
    for (int i = 0; i < 4; i++) {
        nd = (d + 3) % 4;
        nx = x + dx[nd];
        ny = y + dy[nd];
    
        if (0 > nx || nx >= n || 0 > ny || ny >= m || map[nx][ny] == 1 || map[nx][ny] == 2) {
            // Todo : 벽(1)이거나 이미 청소한 장소(2)를 마주쳤을 때

            if (i <= 2) {
                // Todo : 3번까지는 계속 방향을 돌린다.
                d = nd;
                continue;
            }

            // Todo : 4번째
            // 후진했을 때의 좌표와 방향 정보
            int md = (nd + 2) % 4;
            int mx = x + dx[md];
            int my = y + dy[md];
    
            if (map[mx][my] == 2) {
                // Todo : 이미 청소한 장소
                q.add(new Pair(mx, my, nd));
            } else if (map[mx][my] == 1) {
                // Todo : 
                isCompleted = true;
                break;
            }
        }
            
        else if (map[nx][ny] == 0) {
            // Todo : 아직 청소하지 않은 장소(0) 이면 청소한다.
            ...
        }
    }
}      
~~~

* 다음으로 갈 곳이 아직 청소하지 않은 장소(0)일 때의 로직
    1. 청소한다 : 장소의 상태를 2로 바꾼다.
    2. 리스트에 전진한 장소와 바뀐 방향 정보를 추가한다.
~~~
else if (map[nx][ny] == 0) {
    // Todo : 아직 청소 안 한 장소(0) 이면 청소한다.
    map[nx][ny] = 2;
    q.add(new Pair(nx, ny, nd));
    break;
}
~~~
      
---
Link : [BOJ 14503](https://www.acmicpc.net/problem/14503 "로봇청소기")
