Python Linked List
===================


Linked Lists: Attempt
----------------------

    .. code-block:: python

        from __future__ import annotations
        from typing import Optional
        from typing import Iterable


        class Node:
            def __init__(self, value: int, ref: Optional[Node] = None) -> None:
                self.value = value
                self.ref = ref

            def __repr__(self) -> str:
                return f"<Node(value={self.value}, ref={self.ref})>"

            def __str__(self) -> str:
                return str(self.value)


        class SingleLinkedList:
            def __init__(self, iterable: Iterable[int] = None) -> None:
                self.head: Optional[Node] = None
                if iterable:
                    self.extend(iterable)

            def __str__(self) -> str:
                # By nature a linked list is not aware of all of its nodes, so we must iterate
                # all node references.
                start = self.head
                s = ""
                while start:
                    s += "-> " + str(start)
                    start = start.ref
                return "[" + s + "]"

            def append(self, value: int) -> None:
                """
                Register a new node into the linked list, this node is
                appended to the tail of the linked list.
                :param value: The integer to add
                :return: `None`
                """
                if self.head is None:
                    self.head = Node(value)
                    return

                current = self.head
                while current.ref:
                    current = current.ref
                current.ref = Node(value)

            def insert_left(self, value: int) -> None:
                """
                Insert the new value to the left of the linked list, this
                will push the current head forward once and register a
                new node with `value` at the current head.
                :param value: integer to insert at the head of the linked list.
                :return: None
                """
                if self.head is None:
                    self.head = Node(value)
                    return
                new = Node(value, ref=self.head)
                self.head = new

            def extend(self, elements: Optional[Iterable[int]] = None):
                """
                Extends the linked list by appending elements from the iterable.
                :param elements:
                :return:
                """
                for element in elements:
                    self.append(element)

            def clear(self) -> None:
                """
                Erases all references from head in the linked list.
                :return: None
                """
                self.head = None

            def reverse(self) -> None:
                """
                Reverses the linked list in place.
                :return: None
                """
                previous, current = None, self.head
                while current:
                    after = current.ref  # store the next node temporarily
                    current.ref = previous  # set the next node to the previous one
                    previous = current  #
                    current = after  #
                self.head = previous

        #  10 -- 5 -- 8 --
        #  5 -- 8 -- 10 --
        #

        # ---------------------------
        linked = SingleLinkedList((5, 10, 15))
        linked.append(100)
        linked.append(250)
        linked.append(500)
        linked.insert_left(55)
        print(linked)
        linked.reverse()
        print(linked)
        linked.clear()
        print(linked)