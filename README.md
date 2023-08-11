import tkinter as tk
import numpy as np
# 定义棋盘上的空、黑、白三个状态
EMPTY = 0
BLACK = 1
WHITE = -1
# 定义最大最小值
ROWS = 15 #行
COLS = 15 #列
CELL_SIZE = 38 #棋盘总体大小
#制作棋盘
board = [[0] * COLS for _ in range(ROWS)]
#1 代表玩家先手，-1 代表AI先手
player = 1
#操作记录列表
history = []
#玩家落子功能
def play(row, col):
    global player, history
    if board[row][col] == 0:#条件;如果棋盘若为空
        player = 1
        board[row][col] = player#则棋盘更新为玩家落子数据
        history.append((row, col, player))#将玩家的落子数据添加到history中-保存 #乱加会导致黑白子错乱 #删除会导致下黑子之后白子无响应
        draw_piece(row, col, player)#玩家落子功能--删除则棋盘无法落子
        has_winner = check_winner(row, col)  # 胜利判断
        if has_winner:  # 人类玩家落子
            game_over(player)
        else:
            player = -player
            if player == -1:
                AI_move()
# 走法生成函数
def find_best_move(board):
    player = -1
    move = []
    for i in range(len(board)):
        for j in range(len(board[i])):
            if board[i][j] == 0:
                move.append((i, j, player))
    return move
def AI_scores():
    scores= []
    move = find_best_move(board)
    for pos in move:
        test_board = [row[:] for row in board]
        test_board[pos[0]][pos[1]] = -1  # 当前玩家是-1
        score = calculate_score(test_board)  # 进行得分计算
        calculate_score(test_board)
        scores.append(score)

    return scores
def AI_scores1():
    scores = AI_scores()
    return scores
def AI_move():
        import numpy as np
        score1 = AI_scores1()
        max_score1 = max(score1)
        max_count = score1.count(max_score1)
        max_indices = [i for i, value in enumerate(score1) if value == max_score1]
        print("最大值出现次数为：",max_count)
        print("最大值索引分别为：",max_indices)
        max_index = np.argmax(score1)  # 返回最大值位置的索引
        print(max_index)
        move = find_best_move(board)
        element = move[max_index]
        print(element)
        row,col,player = element
        board[row][col] = player
        history.append((row, col, player))
        draw_piece(row, col, player)
        has_winner = check_winner(row, col)  # 胜利判断
        if has_winner:
            game_over(player)
        else:
            play(row,col)

        return max_indices
# 棋型评分表-AI棋局评分
score_table = {
    # 下一步获胜  连五
    "-1-1-1-1-1-1":150,
    "0-1-1-1-1-1":150,
    "1-1-1-1-1-1":150,
    "-1-1-1-1-11":150,
    "-1-1-1-1-10":150,
    # 防止敌方下一步获胜
    "-11111-1": 80,
    "1-11110": 80,
    "01-1110": 80,
    "01-1111": 80,
    "11-1110": 80,
    "11-1111": 80,
    "011-110": 80,
    "011-111": 80,
    "111-110": 80,
    "111-111": 80,
    "0111-11": 80,
    "-11-1111": 80,
    "-1111-11": 80,
    "1-1111-1": 80,
    "11-111-1": 80,
    "111-11-1": 80,
    "-111-111": 80,
    # 下一步制造必胜棋局
    "0-1-1-1-10":70,
    "0-1-1-100":30,
    #破环敌方造必胜棋 白-1
    "0-11110":80,
    "0111-10":80,
    "01101-1":30,
    "-110110": 30,
    "-111010":30,
    "01011-1": 30,
    #其他
    "01-1-1-1-1":30,
    "0-1-1-1-11":30,
    "00-1100":10,
    "001-100":10,
    "00-1-100":10,
    "0011-10":15,
    "01-1100":15,
    "00-1110":15,
}
# 评分函数
def calculate_score(test_board):
    global player, history
    score = 0
    # 水平方向
    for i in range(15):
        for j in range(10):
            line = ''.join(str(test_board[i][j+k]) for k in range(6)) #line为棋型评分表值
            score += score_table.get(line, 0)
            #if score_table.get(line):
                #print("水平方向阵型:", line, "得分:", score)
    # 垂直方向
    for i in range(10):
        for j in range(15):
            line = ''.join(str(test_board[i+k][j]) for k in range(6))
            score += score_table.get(line, 0)
            #if score_table.get(line):
                #print("垂直方向阵型:", line, "得分:", score)
    # 正斜方向
    for i in range(10):
        for j in range(10):
            line = ''.join(str(test_board[i+k][j+k]) for k in range(6))
            score += score_table.get(line, 0)
            #if score_table.get(line):
                #print("正斜方向阵型:", line, "得分:", score)
    # 反斜方向
    for i in range(10):
        for j in range(4, 15):
            line = ''.join(str(test_board[i+k][j-k]) for k in range(6))
            score += score_table.get(line, 0)
            #if score_table.get(line):
                #print("反斜方向阵型:", line, "得分:", score)

    #print(score)
    return score
def draw_piece(row, col, player):#棋子样式功能实现
    color = 'black' if player == 1 else 'white'#棋子黑白两种颜色设置
    x = col * CELL_SIZE + CELL_SIZE // 2 #显示棋子在棋盘的位置，调整会导致棋子与棋盘格子出现错位：左右错位
    y = row * CELL_SIZE + CELL_SIZE // 2 #显示棋子在棋盘的位置，调整会导致棋子与棋盘格子出现错位：上下错位
    canvas.create_oval(x - 12, y - 12, x + 12, y + 12, fill=color)#创建画布
#判断棋局是否胜利
def check_winner(row,col):
    directions = ((1, 0), (0, 1), (1, 1), (1, -1))
    for dr, dc in directions:
        count = 1
        r, c = row, col
        while count < 5:
            r += dr
            c += dc
            if r < 0 or r >= ROWS or c < 0 or c >= COLS or board[r][c] != player:
                break
            count += 1
        r, c = row, col
        while count < 5:
            r -= dr
            c -= dc
            if r < 0 or r >= ROWS or c < 0 or c >= COLS or board[r][c] != player:
                break
            count += 1
        if count >= 5:
            return True
    return False
#获胜显示功能实现
def game_over(player):
    color = '玩家' if player == 1 else 'AI'
    canvas.create_text(CELL_SIZE * COLS // 2, CELL_SIZE * ROWS // 2, text=color + '胜利！', fill='red', font=('Microsoft Yahei', 24, 'bold'))
    #canvas.unbind('<Button-1>')#连五子停止下棋
#棋盘正常显示--若删除则棋盘变白 #画棋盘
def create_board():
    for r in range(ROWS):
        for c in range(COLS):
            canvas.create_rectangle(c * CELL_SIZE, r * CELL_SIZE, (c + 1) * CELL_SIZE, (r + 1) * CELL_SIZE, outline='gray')
#重新开始功能实现
def restart_game():
    global board, player, history,row,col
    board = [[0] * COLS for _ in range(ROWS)]
    player = 1
    history = []
    canvas.delete("all")
    create_board()
#程序导入
root = tk.Tk()
#程序标题
root.title('五子棋')
#棋盘大小
canvas = tk.Canvas(root, width=750, height=568, bg='white')
canvas.pack()
create_board()
canvas.bind('<Button-1>', lambda event: play(event.y // CELL_SIZE, event.x // CELL_SIZE))
#棋盘实现
root.mainloop()
