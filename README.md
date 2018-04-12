파이썬으로 스크롤(Scroll) 및 페이징(Paging) 기능 구현하기 [(원본)](https://github.com/mingrammer/python-curses-scroll-example)
===================================================================
원작자 : [MinJae Kwon](https://github.com/mingrammer) [ref](https://mingrammer.com/how-to-implement-the-scroll-and-paging-in-python-curses/)</br>
재구현 : [Kim MinSu](https://github.com/alstn2468)
-------------------------------------------------
## 스크롤 (Scroll)
- - -
전체 리스트에서의 현재 보여지는 윈도우의 최상단 위치와 현재 커서 위치를 기준으로</br>
다음 커서의 위치를 계산하여 커서를 이동시키는 것이다. 이해하기 쉽게 실제 구현한 코드를 살펴보자.
### 구현
- - -
다음은 TUI 애플리케이션 실행 후 사용자로부터 키보드 입력을 받는 부분이다.</br>
`KEY_UP`과 `KEY_DOWN` 입력을 받게 되면 스크롤을 수행하는 `scroll 메서드`를 `실행`하게된다. (UP=1, DOWN=-1)

```
# 사용자 입력을 대기하며 입력값에 따라 해당되는 메서드를 실행함
def input_stream(self):
    while True:
        self.display()

        ch = self.window.getch()
        if ch == curses.KEY_UP:
            self.scroll(self.UP)
        elif ch == curses.KEY_DOWN:
            self.scroll(self.DOWN)
        elif ch == curses.KEY_LEFT:
            self.paging(self.UP)
        elif ch == curses.KEY_RIGHT:
            self.paging(self.DOWN)
        elif ch == curses.ascii.ESC:
            break
```
- - -

스크롤을 구현할 때 고려해야할 점
> 현재 커서가 현재 윈도우의 최상단 혹은 최하단에 위치하면서 커서와 윈도우가 모두 움직이는 경우
> 현재 커서가 현재 윈도우의 중간에 위치해 커서만 움직이는 경우
- - -

```
max_lines: 한 번에 볼 수 있는 최대 항목의 갯수
current: 현재 보여지는 윈도우 기준 현재 커서 위치
top: 리스트에서의 현재 윈도우의 최상단 라인의 위치
bottom: 커서가 위치할 수 있는 최하단 라인의 위치

# 방향에 따른 다음 라인 커서 위치 계산
def scroll(self, direction):
    next_line = self.current + direction

    # 윈도우 스크롤 업
    # 현재 커서가 윈도우의 상단에 위치하나, 윈도우의 상단 라인이
    # 최상단에 닿지 않았기 때문에 윈도우 스크롤 업이 가능
    if (direction == self.UP) and (self.top > 0 and self.current == 0):
        self.top += direction
        return
    # 윈도우 스크롤 다운
    # 다음 커서가 현재 윈도우의 하단에 위치하나, 커서의 절대 위치가
    # 아직 최하단까지 도달하진 않았기 때문에 윈도우 스크롤 다운이 가능
    if (direction == self.DOWN) and (next_line == self.max_lines) and (self.top + self.max_lines < self.bottom):
        self.top += direction
        return
    # 스크롤 업
    # 현재 커서가 최상단보다 아래에 있으므로 스크롤 업이 가능
    if (direction == self.UP) and (self.top > 0 or self.current > 0):
        self.current = next_line
        return
    # 스크롤 다운
    # 다음 커서가 현재 윈도우의 하단보다 위에 있으며, 커서의 절대 위치가
    # 아직 최하단까지 도달하진 않았으므로 스크롤 다운이 가능
    if (direction == self.DOWN) and (next_line < self.max_lines) and (self.top + next_line < self.bottom):
        self.current = next_line
        return
```

### DEMO
- - -
> 스크롤 업 : ↑
> 스크롤 다운 : ↓

[위로](#Scroll)
</br>

## 페이징 (Paging)
- - -
스크롤은 커서 위치를 조정하면서 동작하는 반면, 페이징은 윈도우의 상단 라인 (`top` 변수)의 위치를 조정하면서 동작한다.</br>
페이징을 구현할 때에는 한가지 주의 해야할 부분이 있는데, 페이징을 하다가 `마지막 페이지에 도달했을 때`</br>
현재 커서가 `마지막 페이지`에 나타나는 항목의 리스트보다 `아래`에 위치하는 경우, 이 커서 위치를 `재조정`해줘야 한다는 것이다.

### 구현
