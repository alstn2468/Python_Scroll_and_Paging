파이썬으로 스크롤(Scroll) 및 페이징(Paging) 기능 구현하기
===================================================================
원작자 : [MinJae Kwon](https://github.com/mingrammer) / [ref](https://mingrammer.com/how-to-implement-the-scroll-and-paging-in-python-curses/)</br>
재구현 : [Kim MinSu](https://github.com/alstn2468)
-------------------------------------------------
## 스크롤 (Scroll)
- - -
전체 리스트에서의 현재 보여지는 윈도우의 최상단 위치와 현재 커서 위치를 기준으로</br>
다음 커서의 위치를 계산하여 커서를 이동시키는 것이다. 이해하기 쉽게 실제 구현한 코드를 살펴보자.
### 구현
- - -
다음은 TUI 애플리케이션 실행 후 사용자로부터 키보드 입력을 받는 부분이다.</br>
`KEY_UP`과 `KEY_DOWN` 입력을 받게 되면 스크롤을 수행하는 `scroll` 메서드를 `실행`하게된다. (UP=1, DOWN=-1)

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
> 현재 커서가 현재 윈도우의 최상단 혹은 최하단에 위치하면서 커서와 윈도우가 모두 움직이는 경우</br>
> 현재 커서가 현재 윈도우의 중간에 위치해 커서만 움직이는 경우

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
> 스크롤 업   : ↑</br>
> 스크롤 다운 : ↓

</br>

[위로](#파이썬으로-스크롤scroll-및-페이징paging-기능-구현하기-원본)

</br>

## 페이징 (Paging)
- - -
스크롤은 커서 위치를 조정하면서 동작하는 반면, 페이징은 윈도우의 상단 라인 (`top` 변수)의 위치를 조정하면서 동작한다.</br>
페이징을 구현할 때에는 한가지 주의 해야할 부분이 있는데, 페이징을 하다가 `마지막 페이지에 도달했을 때`</br>
현재 커서가 `마지막 페이지`에 나타나는 항목의 리스트보다 `아래`에 위치하는 경우, 이 커서 위치를 `재조정`해줘야 한다는 것이다.

### 구현
- - -
키보드 입력을 받는 부분에서 `KEY_LEFT`와 `KEY_RIGHT` 입력을 받게 되면 페이징을 수행하는 `paging` 메서드를 실행하게된다.

```
def paging(self, direction):
    # 윈도우의 상단 위치값과 현재 커서 위치로 현재 페이지와 다음 페이지를 계산
    current_page = (self.top + self.current) // self.max_lines
    next_page = current_page + direction

    # 마지막 페이지에 도달 했을 때 현재 커서가 마지막 페이지에 나타나는 항목의 리스트보다 아래에 있는 경우,
    # 현재 커서를 마지막 페이지 리스트의 마지막 항목 위치로 조정
    if next_page == self.page:
        self.current = min(self.current, self.bottom % self.max_lines - 1)

    # 페이지 업
    # 현재 페이지가 첫 페이지가 아닌 경우, 페이지 업이 가능
    # 윈도우 상단의 위치는 음수가 될 수 없기 때문에 음수가 될 경우 0으로 조정
    if (direction == self.UP) and (current_page > 0):
        self.top = max(0, self.top - self.max_lines)
        return

    # 페이지 다운
    # 현재 페이지가 마지막 페이지가 아닌 경우, 페이지 다운이 가능
    if (direction == self.DOWN) and (current_page < self.page):
        self.top += self.max_lines
        return
```

### DEMO
> 페이징 업   : ←</br>
> 페이징 다운 : →

</br>

[위로](#파이썬으로-스크롤scroll-및-페이징paging-기능-구현하기-원본)/[스크롤](#스크롤-scroll)

</br>

## LICENSE
- - -
MIT LICENSE</br>
[(원본)](https://github.com/mingrammer/python-curses-scroll-example)
