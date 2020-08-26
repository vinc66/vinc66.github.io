---
title: Data_structures_linked_list
date: 2017-03-29 20:45:42
tags:
    data_structures
---

---
#### 1.链表定义

    链表即是由节点（Node）组成的线性集合，每个节点可以利用指针指向其他节点。它是一种包含了多个节点的、能够用于表示序列的数据结构。
    单向链表: 链表中的节点仅指向下一个节点，并且最后一个节点指向空。
    双向链表: 其中每个节点具有两个指针 p、n，使得 p 指向先前节点并且 n 指向下一个节点；最后一个节点的 n 指针指向 null。
    循环链表：每个节点指向下一个节点并且最后一个节点指向第一个节点的链表。
    时间复杂度:
    索引: O(n)
    搜索: O(n)
    插入: O(1)
    移除: O(1)

---

#### 2.code

---


    //俩链表相加结果逆序放在一个新的链表
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode l11 = l1;
        ListNode l22 = l2;
        ListNode lo = new ListNode(0);
        ListNode head = lo;
        int sum = 0;
        while (l11 != null || l22 != null) {
            sum /= 10;
            if (l11 != null) {
                sum += l11.val;
                l11 = l11.next;
            }
            if (l22 != null) {
                sum += l22.val;
                l22 = l22.next;
            }
            head.next = new ListNode(sum % 10);
            head = head.next;
        }
        if (sum / 10 == 1)
            head.next = new ListNode(1);
        return lo.next;
    }

    //删除链表中指定的元素
    public void deleteNode(ListNode node) {
        node.val = node.next.val;
        node.next = node.next.next;
    }

    //链表逆序展示
    public ListNode reverseList(ListNode head) {
        if (head == null) return head;
        ListNode newList = new ListNode(0);
        while (head != null) {
            ListNode tmp = head;
            head = tmp.next;
            tmp.next = newList;
            newList = tmp;
        }
        return newList;
    }

    /**
     * 回文结构  123321 正反读一样
     * 一个读的快 一个读的慢，
     * 栈先进后出
     *
     * @param head
     * @return
     */
    public boolean isPalindrome(ListNode head) {
        ListNode s = head;
        ListNode f = head;
        Stack<Integer> stack = new Stack();

        while (f != null && f.next != null) {
            stack.push(s.val);
            f = f.next.next;
            s = s.next;
        }
        //如果只有一个数
        if (f != null)
            s = s.next;
        while (s.next != null) {
            if (s.next.val != stack.pop()) return false;
            s = s.next;
        }
        return true;
    }

    //链表排序
    public ListNode mergeKLists(ListNode[] lists) {
        ListNode lo = new ListNode(0);
        ListNode head = lo;

        for (int i = 0; i < lists.length; i++) {
            for (int j = 0; j < lists.length - 1; j++) {
                if (lists[j].val < lists[j + 1].val) {
                    ListNode tmp = null;
                    tmp = lists[j];
                    lists[j] = lists[j + 1];
                    lists[j + 1] = tmp;
                }
            }
        }
        for (ListNode l : lists) {
            head.next = l;
            head = head.next;
        }
        return lo.next;
    }
    class ListNode {
        int val;
        ListNode next;

        ListNode(int x) {
            val = x;
        }

    }


---

