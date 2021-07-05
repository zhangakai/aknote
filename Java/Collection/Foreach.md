#各种循环的区别

----------------------------------------------------

Collection.foreach(), Iterator.foreach(), Stream.foreach(), for( : )

    以ArrayList为例
    Collection.foreach():


    Iterator.foreach():
    Iterator var2 = list.iterator();
        
    while(var2.hasNext()) {
        int i = (Integer)var2.next();
        System.out.println(i);
    }
    ArrayList内部实现了自己的Iterator
    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        Itr() {}

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
    }

    可以看到有两个if分支

    
    