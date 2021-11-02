import numpy as np
import time
i = 0
N = 3
goal = [i+1 for i in range(N*N)]
goal[N*N-1] = 0

class Board:

    def __init__(self, tiles):
        self.Array = np.array(tiles).reshape(N,N)        

    def hamming(self):
        temp = N*N-1-sum(sum(self.Array == Board(goal).Array))
        if sum(sum(np.argwhere(self.Array == 0))) == 2*(N-1):            
            return temp+1
        else:
            return temp


    def manhattan(self):
        manhattan_i = []
        for i in range(N*N-1):
            manhattan_i.append(sum(sum(abs(np.argwhere(self.Array == i+1)-np.argwhere(Board(goal).Array == i+1)))))
        return sum(manhattan_i) 


    def equals(self,y):
        return sum(sum(self.Array == y.Array)) == N*N


    def neighbours(self):
        location = np.argwhere(self.Array==0)[0]
        x = location[0]
        y = location[1]
        neighbors = []
        copies= [self.Array.copy() for i in range(4)]
        if x != 0:
            copies[0][x][y] = self.Array[x-1][y]
            copies[0][x-1][y] = 0
            neighbors.append(Board(copies[0]))
        if x != N-1:
            copies[1][x][y] = self.Array[x+1][y]
            copies[1][x+1][y] = 0
            neighbors.append(Board(copies[1]))
        if y != 0:
            copies[2][x][y] = self.Array[x][y-1]
            copies[2][x][y-1] = 0
            neighbors.append(Board(copies[2]))
        if y != N-1:
            copies[3][x][y] = self.Array[x][y+1]
            copies[3][x][y+1] = 0        
            neighbors.append(Board(copies[3]))
        return neighbors

    def toString(self):
        for i in range(N):
            for j in range(N):
                if self.Array[i][j] != 0:
                    print(str(self.Array[i][j])+'  ',end="")
                else:
                    print('   ',end="")
            print('\n')

class State:
    def __init__(self,board,move,pre):
        self.Board = board
        self.Move = move
        self.Pre = pre    
        
class MinPQ:
    def __init__(self):
        self.states = []
        self.states.append(0)
        self.n = 0
    def insert(self, newstate):
        self.n = self.n+1
        self.states.append(newstate)
        self.swim(self.n)
    def delMin(self):
        Min = self.states[1]
        self.exch(1, self.n)
        self.n = self.n-1
        del self.states[self.n + 1]
        self.sink(1)
        return Min
        
    def swim(self,k):
        if k <= 1:
            return
        while (k > 1) & (self.states[int(k/2)].Board.manhattan() > self.states[k].Board.manhattan()):
                self.exch(int(k/2) , k)
                k = int(k/2)
                if k <= 1:
                    return

    def sink(self,k):
        while (2*k <= self.n):
            j = 2*k
            if (j < self.n):
                if (self.states[j].Board.manhattan() > self.states[j+1].Board.manhattan()):
                    j = j+1
            if not self.states[k].Board.manhattan() > self.states[j].Board.manhattan():
                break
            self.exch(k, j)
            k = j
    
    def exch(self,i,j):
        t = self.states[i]
        self.states[i] = self.states[j]
        self.states[j] = t


class Solver:
    def __init__(self, board):
        self.initial_state = State(board,0,None)
        self.pq = MinPQ()
        self.pq.insert(self.initial_state)
        if self.isSolvable(): 
            while self.pq.states[1].Board.manhattan() != 0:
                dequeue = self.pq.delMin()
                for neighbor in dequeue.Board.neighbours():
                    if dequeue.Pre is None:
                        self.pq.insert(State(neighbor,dequeue.Move+1,dequeue))
                    elif not neighbor.equals(dequeue.Pre.Board):
                        Flag = 1
                        for state in self.pq.states:
                            if state == 0:
                                pass
                            elif state.Board.equals(neighbor):
                                Flag = 0
                        if Flag == 1:    
                            self.pq.insert(State(neighbor,dequeue.Move+1,dequeue))
                            
        else:
            print("It is not solvable!")
    def isSolvable(self):
        List = list(self.initial_state.Board.Array.reshape(1,N*N)[0])
        k = 0
        for i in range(1,N*N-1):
            for j in range(0,i):
                if List[j] > List[i]:
                    k = k+1
        return k%2 == 0
    def moves(self):
        return self.pq.states[1].Move
    def toString(self):
        Move_list = []
        Move_list.append(self.pq.states[1])
        while Move_list[-1].Pre != None:
            Move_list.append(Move_list[-1].Pre)
        Move_list.reverse()
        for each in Move_list:
            each.Board.toString()
            print('\n')

if __name__ == '__main__':
    a = [4,5,1,8,3,7,6,2,0]
    test = Board(a)
    solve = Solver(test)
    solve.toString()
    b = [1,2,3,4,5,6,7,8,0]
    test = Board(b)
    solve = Solver(test)
![1](https://user-images.githubusercontent.com/93483043/139594416-ffc3bccd-fa04-45fd-900f-7b4d74c107c3.PNG)
![2](https://user-images.githubusercontent.com/93483043/139594417-2e112379-b533-49a4-b9b0-e4c39ca7e044.PNG)
![3](https://user-images.githubusercontent.com/93483043/139594418-fd609939-d19e-4ecf-ba4a-efb09099dd9d.PNG)
![4](https://user-images.githubusercontent.com/93483043/139594419-bbea77e2-7491-433a-bc6b-69189a5c28bf.PNG)
![5](https://user-images.githubusercontent.com/93483043/139594420-1ba3997e-6bb6-4536-8dbe-22aa6fbe70a1.PNG)
![6](https://user-images.githubusercontent.com/93483043/139594421-9a307fb7-7041-437e-893e-898e580b0192.PNG)
![7](https://user-images.githubusercontent.com/93483043/139594422-9d48c86c-4b11-4bcf-8750-25b05a8af38e.PNG)
