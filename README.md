# Robot-Collisions

There are n 1-indexed robots, each having a position on a line, health, and movement direction.

You are given 0-indexed integer arrays positions, healths, and a string directions (directions[i] is either 'L' for left or 'R' for right). All integers in positions are unique.

All robots start moving on the line simultaneously at the same speed in their given directions. If two robots ever share the same position while moving, they will collide.

If two robots collide, the robot with lower health is removed from the line, and the health of the other robot decreases by one. The surviving robot continues in the same direction it was going. If both robots have the same health, they are both removed from the line.

Your task is to determine the health of the robots that survive the collisions, in the same order that the robots were given, i.e. final health of robot 1 (if survived), final health of robot 2 (if survived), and so on. If there are no survivors, return an empty array.

Return an array containing the health of the remaining robots (in the order they were given in the input), after no further collisions can occur.

Note: The positions may be unsorted.

class Robot:
    def __init__(self, label, position, health, direction):
        self.label = label
        self.position = position
        self.health = health
        self.direction = direction
        self.prev = None
        self.next = None

class RobotList:
    def __init__(self):
        self.head = None
        self.tail = None

    def append(self, robot):
        if not self.head:
            self.head = self.tail = robot
        else:
            self.tail.next = robot
            robot.prev = self.tail
            self.tail = robot

    def remove(self, robot):
        if robot.prev:
            robot.prev.next = robot.next
        if robot.next:
            robot.next.prev = robot.prev
        if robot == self.head:
            self.head = robot.next
        if robot == self.tail:
            self.tail = robot.prev

    def collide(self, left, right):
        if left.health > right.health:
            left.health -= 1
            self.remove(right)
            right = right.next
        elif right.health > left.health:
            right.health -= 1
            self.remove(left)
            left = left.prev if left.prev else right
        else:
            self.remove(left)
            self.remove(right)
            left = left.prev if left.prev else right.next
            right = right.next if right.next else None

        return left, right

    def collide_all(self):
        curr = self.head
        while curr:
            if curr.prev and curr.prev.direction == 'R' and curr.direction == 'L':
                _, curr = self.collide(curr.prev, curr)
            elif curr.next and curr.direction == 'R' and curr.next.direction == 'L':
                curr, _ =  self.collide(curr, curr.next)
            else:
                curr = curr.next

class Solution:
    def survivedRobotsHealths(self, positions: List[int], healths: List[int], directions: str) -> List[int]:
        robots = [Robot(i + 1, pos, health, dir) for i, (pos, health, dir) in enumerate(zip(positions, healths, directions))]
        robots.sort(key=lambda robot: robot.position)

        robot_list = RobotList()
        for robot in robots:
            robot_list.append(robot)
        
        robot_list.collide_all()

        remaining = []
        curr = robot_list.head
        while curr:
            remaining.append(curr)
            curr = curr.next
        remaining.sort(key=lambda robot: robot.label)
        return [robot.health for robot in remaining]
