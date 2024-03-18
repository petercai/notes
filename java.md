#  **singly linked list** in Java. 

A linked list consists of nodes, where each node contains data and a reference to the next node. Here’s an example implementation:

```java
public class LinkedList {
    // Node class representing individual elements in the list
    static class Node {
        int data; // Data stored in the node
        Node next; // Reference to the next node

        Node(int d) {
            data = d;
            next = null;
        }
    }

    Node head; // Head of the linked list

    // Method to insert a new node at the end of the list
    public static LinkedList insert(LinkedList list, int data) {
        Node new_node = new Node(data);
        if (list.head == null) {
            // If the list is empty, set the new node as the head
            list.head = new_node;
        } else {
            // Otherwise, traverse to the last node and add the new node
            Node last = list.head;
            while (last.next != null) {
                last = last.next;
            }
            last.next = new_node;
        }
        return list;
    }

    // Method to print the linked list
    public static void printList(LinkedList list) {
        Node currNode = list.head;
        System.out.print("LinkedList: ");
        while (currNode != null) {
            System.out.print(currNode.data + " ");
            currNode = currNode.next;
        }
    }

    public static void main(String[] args) {
        LinkedList list = new LinkedList();
        list = insert(list, 1);
        list = insert(list, 2);
        list = insert(list, 3);
        list = insert(list, 4);
        list = insert(list, 5);
        printList(list);
    }
}
```

In this example:

- We define a `Node` class to represent individual elements (nodes) in the linked list.
- The `insert` method adds a new node with the given data at the end of the list.
- The `printList` method traverses the list and prints its contents.

When you run this program, it will create a linked list with values 1, 2, 3, 4, and 5, and print them in order. Feel free to modify or extend this basic implementation as needed! 🌟

For more advanced features (e.g., doubly linked lists, deletion, searching), you can explore additional functionalities. [Happy coding! 😊](https://www.geeksforgeeks.org/implementing-a-linked-list-in-java-using-class/) [1](https://www.geeksforgeeks.org/implementing-a-linked-list-in-java-using-class/)[2](https://www.programiz.com/java-programming/linkedlist)[3](https://www.educative.io/blog/data-structures-linked-list-java-tutorial)

# Support the `insertAfter` operation for  `LinkedList`  

Here’s the improved version:

```java
public class LinkedList {
    static class Node {
        int data;
        Node next;

        Node(int d) {
            data = d;
            next = null;
        }
    }

    Node head;

    // Insert a new node after a given node
    public void insertAfter(int nodeData, int newData) {
        Node newNode = new Node(newData);
        Node currentNode = head;

        while (currentNode != null) {
            if (currentNode.data == nodeData) {
                newNode.next = currentNode.next;
                currentNode.next = newNode;
                return; // Insertion done
            }
            currentNode = currentNode.next;
        }

        System.out.println("Node with data " + nodeData + " not found.");
    }

    // Print the linked list
    public void printList() {
        Node currNode = head;
        System.out.print("LinkedList: ");
        while (currNode != null) {
            System.out.print(currNode.data + " ");
            currNode = currNode.next;
        }
    }

    public static void main(String[] args) {
        LinkedList list = new LinkedList();
        list.head = new Node(1);
        list.insertAfter(1, 2); // Insert 2 after 1
        list.insertAfter(2, 3); // Insert 3 after 2
        list.insertAfter(2, 4); // Insert 4 after 2
        list.printList(); // Output: 1 2 4 3
    }
}
```

In this improved version:

- We’ve added the `insertAfter` method, which takes the value of an existing node (`nodeData`) and the data for the new node (`newData`).
- It searches for the node with the specified value (`nodeData`) and inserts the new node after it.
- If the specified node is not found, it prints a message indicating that.

Now you can use the `insertAfter` method to add nodes after specific values in your linked list. [Feel free to adapt or extend this implementation as needed! 🌟](https://stackoverflow.com/questions/33973052/java-linked-list-insertafter-method) [1](https://stackoverflow.com/questions/33973052/java-linked-list-insertafter-method)[2](https://codereview.stackexchange.com/questions/104940/inserting-a-node-to-a-given-position-in-a-linked-list)[3](https://codereview.stackexchange.com/questions/88078/better-method-structure-for-insertion-in-doubly-linked-list)