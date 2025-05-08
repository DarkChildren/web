---
title: "Advanced Structures"
sidebar_label: "Advanced Structures"
---

## Array manipulation

### Encontrar una posición vacía correctamente

Este ejemplo muestra cómo encontrar una posición vacía en un arreglo usando prácticas estándar de programación.

```pawn
new
    gMyArray[10];

stock FindEmptySlot()
{
    new
        i = 0;
    while (i < sizeof (gMyArray) && gMyArray[i])
    {
        i++;
    }
    if (i == sizeof (gMyArray)) return -1;
    return i;
}
```

Este ejemplo básico asume que una posición del arreglo está vacía si su valor es 0. El bucle recorre todos los valores del arreglo (también podría hacerse con una constante) mientras los valores no sean 0. Cuando encuentra uno que sea 0, la condición del while fallará y el bucle terminará sin usar break, lo cual es una práctica común pero desaconsejada en situaciones como esta. Esta función también devuelve -1 si no se encuentra una posición libre, lo cual debe verificarse desde el otro lado. Más comúnmente, se usaría el ID encontrado de inmediato:

```pawn
MyFunction()
{
    new
        i = 0;
    while (i < sizeof (gMyArray) && gMyArray[i])
    {
        i++;
    }
    if (i == sizeof (gMyArray))
    {
        printf("No free slot found");
        return 0;
    }
    printf("Slot %d is empty", i);
    // Use the found slot in your code for whatever
    return 1;
}
```

Obviamente, deberías reemplazar gMyArray[i] con tu propia condición de posición ocupada.

### Listas

#### Introducción

Las listas son un tipo de estructura muy útil; básicamente son un arreglo donde el siguiente dato relevante es apuntado por el dato anterior.

Ejemplo:

Supón que tienes el siguiente arreglo:

```pawn
3, 1, 64, 2, 4, 786, 2, 9
```

Sin quisieras ordenarlo obtendrías:

```pawn
1, 2, 2, 3, 4, 9, 64, 786
```

Pero si quisieras mantener el orden original de los datos y aun así conocer los números en orden (por alguna razón), ¿cómo tendrías los números en dos órdenes al mismo tiempo? Aquí es donde las listas son útiles. Para construir una lista de estos datos necesitas convertir el arreglo en uno 2D, donde la segunda dimensión tiene 2 celdas: la primera contiene el número original y la otra el índice del siguiente número más grande. También necesitas una variable separada para guardar el índice del número más pequeño. El nuevo arreglo sería:

```pawn
start = 1
3, 1, 64, 2, 4, 786, 2, 9
4, 3, 5,  6, 7, -1,  0, 2
```

El índice asociado a 786 es -1, lo que indica el final de la lista. Los dos 2 obviamente podrían estar en cualquier orden, el primero es el que aparece primero en la lista ya que es más probable que se encuentre antes.

Otra ventaja de este método de ordenar números es que agregar más es mucho más rápido. Si quisieras agregar otro número 3 a la lista ordenada, tendrías que mover al menos 4 números una posición a la derecha. En una lista, solo agregas el nuevo número al final y modificas un único valor:

```pawn
start = 1
3, 1, 64, 2, 4, 786, 2, 9, 3
8, 3, 5,  6, 7, -1,  0, 2, 4
^ modificar este valor        ^ Apunta al siguiente número más alto
```

Los otros números no se mueven, por lo tanto sus índices no se modifican.
Eliminar un valor es incluso más fácil:

```pawn
start = 1
3, 1, 64, X, 4, 786, 2, 9, 3
8, 6, 5,  6, 7, -1,  0, 2, 4
   ^ cambiado para saltar el valor eliminado
```

Aquí, se ha eliminado el primer 2 y se ha actualizado el índice del número que lo apuntaba (el 1) para que apunte al número al que apuntaba el 2 eliminado. Técnicamente no se borra ni el número ni el puntero, pero ya no se puede alcanzar esa posición, así que efectivamente está eliminado.


#### Tipos

Las listas anteriores eran listas simples. También se pueden tener listas dobles, donde cada valor apunta al siguiente y al anterior. Estas suelen tener un puntero al final de la lista para poder recorrerla hacia atrás:

```pawn
start = 1
end = 5
value: 3, 1,  64, 2, 4, 786, 2, 9, 3
next:  8, 3,  5,  6, 7, -1,  0, 2, 4
last:  6, -1, 7,  1, 8, 2,   3, 4, 0
```

Debes tener cuidado con estas, especialmente cuando hay valores repetidos. Por ejemplo, esto está mal:

```pawn
2,  3, 3
1,  2, -1
-1, 2, 0
```

El puntero de "siguiente" de 2 apunta al 3 en la posición 1, pero el puntero "anterior" de ese 3 no apunta de regreso al 2. Una versión correcta sería:

```pawn
2,  3, 3
1,  2, -1
-1, 0, 2
```

Both of those lists start and end on the end two numbers, the back list in the wrong example started on the middle number.

The other type of list is the looping one where the last value points back to the first. The obvious advantage to this is that you can get to any value from any other value without knowing in advance whether the target is before or after the start point, you just need to be careful not to get into an infinite loop as there's no explicit -1 end point. These lists do still have start points. You can also do double looping lists where you have a next and last list, both of which loop round:

```pawn
start = 1
end = 5
3, 1,  64, 2, 4, 786, 2, 9, 3
8, 3,  5,  6, 7, 1,   0, 2, 4
6, 5,  7,  1, 8, 2,   3, 4, 0
```

#### Listas Mixtas

Las listas mixtas contienen múltiples listas en el mismo arreglo. Por ejemplo, puedes tener una lista ordenada y otra para rastrear espacios libres:

```pawn
sortedStart = 3
unusedStart = 1
value: 34, X, X, 6, 34, 46, X,  54, 23, 25, X,  75, X, 45
sort:  4,        8, 13, 7,      11, 9,  0,      -1,    5
free:      2, 6,            10,             12,     -1
```

Obviously the two lists never interact so both can use the same slot for their next value:

```pawn
sortedStart = 3
unusedStart = 1
value: 34, X, X, 6, 34, 46, X,  54, 23, 25, X,  75, X,  45
next:  4,  2, 6, 8, 13, 7,  10, 11, 9,  0,  12, -1, -1, 5
```

#### Code

Before you start the code you need to decide what sort of list is best suited for your application, this is entirely based on application can't easily be covered here. All these examples are mixed lists, one list for the required values, one for unused slots.

This example shows how to write code for a list sorted numerically ascending.

```pawn
#define NUMBER_OF_VALUES (10)

enum E_DATA_LIST
{
    E_DATA_LIST_VALUE,
    E_DATA_LIST_NEXT
}

new
    gListData[NUMBER_OF_VALUES][E_DATA_LIST],
    gUnusedStart = 0,
    gListStart = -1; // Starts off with no list

// This function initializes the list
List_Setup()
{
    new
        i,
        size = NUMBER_OF_VALUES;
    size--;
    for (i = 0; i < size; i++)
    {
        // To start with all slots are unused
        gListData[i][E_DATA_LIST_NEXT] = i + 1;
    }
    // End the list
    gListData[size][E_DATA_LIST_NEXT] = -1;
}

// This function adds a value to the list (using basic sorting)
List_Add(value)
{
    // Check if there are free slots in the array
    if (gUnusedStart == -1) return -1;
    new
        pointer = gListStart,
        last = -1,
        slot = gUnusedStart;
    // Add the value to the array
    gListData[slot][E_DATA_LIST_VALUE] = value;
    // Update the empty list
    gUnusedStart = gListData[slot][E_DATA_LIST_NEXT];
    // Loop through the list till we get to bigger/same size number
    while (pointer != -1 && gListData[pointer][E_DATA_LIST_VALUE] < value)
    {
        // Save the position of the last value
        last = pointer;
        // Move on to the next slot
        pointer = gListData[pointer][E_DATA_LIST_NEXT];
    }
    // If we got here we ran out of values or reached a larger one
    // Check if we checked any numbers
    if (last == -1)
    {
        // The first number was bigger or there is no list
        // Either way add the new value to the start of the list
        gListData[slot][E_DATA_LIST_NEXT] = gListStart;
        gListStart = slot;
    }
    else
    {
        // Place the new value in the list
        gListData[slot][E_DATA_LIST_NEXT] = pointer;
        gListData[last][E_DATA_LIST_NEXT] = slot;
    }
    return slot;
}

// This function removes a value from a given slot in the array (returned by List_Add)
List_Remove(slot)
{
    // Is this a valid slot
    if (slot < 0 || slot >= NUMBER_OF_VALUES) return 0;
    // First find the slot before
    new
        pointer = gListStart,
        last = -1;
    while (pointer != -1 && pointer != slot)
    {
        last = pointer;
        pointer = gListData[pointer][E_DATA_LIST_NEXT];
    }
    // Did we find the slot in the list
    if (pointer == -1) return 0;
    if (last == -1)
    {
        // The value is the first in the list
        // Skip over this slot in the list
        gListStart = gListData[slot][E_DATA_LIST_NEXT];
    }
    else
    {
        // The value is in the list
        // Skip over this slot in the list
        gListData[last][E_DATA_LIST_NEXT] = gListData[slot][E_DATA_LIST_NEXT];
    }
    // Add this slot to the unused list
    // The unused list isn't in any order so this doesn't matter
    gListData[slot][E_DATA_LIST_NEXT] = gUnusedStart;
    gUnusedStart = slot;
    return 1;
}
```

### Binary Trees

#### Introduction

Binary trees are a very fast method of searching for data in an array by using a very special list system. The most well known binary tree is probably the 20 questions game, with just 20 yes/no questions you can have over 1048576 items. A binary tree, as its name implies, is a type of tree, similar to a family tree, where every item has 0, 1 or 2 children. They are not used for ordering data like a list but sorting data for very efficient searching. Basically you start with an item somewhere near the middle of the ordered list of objects (e.g. the middle number in a sorted array) and compare that to the value you want to find. If it's the same you've found your item, if it's greater you move to the item to the right (not immediately to the right, the item to the right of the middle item would be the item at the three quarter mark), if it's less you move left, then repeat the process.

**Example**

```pawn
1 2 5 6 7 9 12 14 17 19 23 25 28 33 38
```

You have the preceding ordered array and you want to find what slot the number 7 is in (if it's in at all), in this example it's probably more efficient to just loop straight through the array to find it but that's not the point, that method increases in time linearly with the size of the array, a binary search time increases linearly as the array increases exponentially in size. I.e. an array 128 big will take twice as long to search straight through as an array 64 big, but a binary search 128 big will only take one check more than a binary search 64 big, not a lot at all.

If we construct a binary tree from the data above we get: ![Binarytree](https://sampwiki.blast.hk/wiki/Image:Binarytree.GIF)

If you read left to right, ignoring the vertical aspect you can see that the numbers are in order. Now we can try to find the 7.

The start number is 14, 7 is less than 14 so we go to the slot pointed to by the left branch of 14. This brings us to 6, 7 is bigger than 6 so we go right to 9, then left again to 7. This method took 4 comparisons to find the number (including the final check to confirm that we are on 7), using a straight search would have taken 5.

Lets say there is no 7, we would end up with this binary tree: ![Binarytree-7-less](https://sampwiki.blast.hk/wiki/Image:Binarytree-7-less.GIF)

This, unlike the example above, has a single child number (the 9), as well as 2 and 0 child numbers. You only get a perfect tree when there are (2^n)-1 numbers (0, 1, 3, 7, 15, 31 ...), any other numbers will give a not quite full tree. In this case when we get to the 9, where the 7 will be, we'll find there is no left branch, meaning the 7 doesn't exist (it cannot possibly be anywhere else in the tree, think about it), so we return -1 for invalid slot.

#### Balanced and unbalanced

The trees in the examples above are called balanced binary trees, this means as near as possible all the branches are the same length (obviously in the second there aren't enough numbers for this to be the case but it's as near as possible). Constructing balanced trees is not easy, the generally accepted method of constructing almost balanced trees is putting the numbers in in a random order, this may mean you end up with something like this: ![Binarytree-uneven](https://sampwiki.blast.hk/wiki/Image:Binarytree-uneven.GIF)

Obviously this tree is still valid but the right side is much larger than the left, however finding 25 still only takes 7 comparisons in this compared to 12 in the straight list. Also, as long as you start with a fairly middle number the random insertion method should produce a fairly balanced tree. The worst possible thing you can do is put the numbers in in order as then there will be no left branches at all (or right branches if done the other way), however even in this worst case the binary tree will take no longer to search than the straight list.

**Modification**

#### Addition

Adding a value to a binary tree is relatively easy, you just follow the tree through, using the value you want to add as a reference untill you reach an empty branch and add the number there. E.g. if you wanted to add the number 15 to our original balanced tree it would end up on the left branch of the 17. If we wanted to add the number 8 to the second balanced tree (the one without the 7) it would end up in the 7's old slot on the left of the 9.

#### Deletion

Deleting a number from a binary tree can be hard or it can be easy. If the number is at the end of a branch (e.g. 1, 5, 7, 12 etc in the original tree) you simply remove them. If a number only has one child (e.g. the 9 in the second example) you simply move that child (e.g. the 12) up into their position (so 6's children would be 2 and 12 in the new second example with 9 removed). Deletion only gets interesting when a node has two children. There are at least four ways of doing this:

The first method is the simplest computationally. Basically you choose one of the branches (left or right, assume right for this explanation) and replace the node you've removed with the first node of that branch (i.e. the right child of the node you've removed). You then go left through the new branch till you reach the end and place the left branch there. E.g. if you removed the 14 from the original exampe you would end up with 25 taking its place at the top of the tree and 6 attached to the left branch of 17. This method is fast but ends up with very unbalanced trees very quickly.

The second method is to get all the numbers which are children of the node you just removed and rebuild a new binary tree from them, then put the top of that tree into the node you've just removed. This keeps the tree fairly well balanced but is obviously slower.

The third method is to combine the two methods above and rebuild the tree inline, this is more complex to code but keeps the tree balanced and is faster than the second method (though no-where near as fast as the first).

The final menthod listed here is to simply set a flag on a value saying it's not used any more, this is even faster than the first method and maintains the structure but means you can't re-use slots unless you can find a value to replace it with later.
