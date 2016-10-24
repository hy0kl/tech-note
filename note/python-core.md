# python2.5 内核剖析

## 小整数对象池

`Objects/intobject.c`

```
#ifndef NSMALLPOSINTS
#define NSMALLPOSINTS       257
#endif
#ifndef NSMALLNEGINTS
#define NSMALLNEGINTS       5
#endif
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
/* References to small integers are saved in this array so that they
   can be shared.
   The integers that are saved are those in the range
   -NSMALLNEGINTS (inclusive) to NSMALLPOSINTS (not inclusive).
*/
static PyIntObject *small_ints[NSMALLNEGINTS + NSMALLPOSINTS];
#endif

int
_PyInt_Init(void)
{
    PyIntObject *v;
    int ival;
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
    for (ival = -NSMALLNEGINTS; ival < NSMALLPOSINTS; ival++) {
              if (!free_list && (free_list = fill_free_list()) == NULL)                                                                                                   
            return 0;
        /* PyObject_New is inlined */
        v = free_list;
        free_list = (PyIntObject *)v->ob_type;
        PyObject_INIT(v, &PyInt_Type);
        v->ob_ival = ival;
        small_ints[ival + NSMALLNEGINTS] = v;
    }
#endif
    return 1;
}
```

## 大整数对象的单向链表

`Objects/intobject.c`

```c
#define BLOCK_SIZE  1000    /* 1K less typical malloc overhead */
#define BHEAD_SIZE  8   /* Enough for a 64-bit pointer */
#define N_INTOBJECTS    ((BLOCK_SIZE - BHEAD_SIZE) / sizeof(PyIntObject))

struct _intblock {
    struct _intblock *next;
    PyIntObject objects[N_INTOBJECTS];
};

typedef struct _intblock PyIntBlock;

static PyIntBlock *block_list = NULL;
static PyIntObject *free_list = NULL;
```

## 字符串对象

interned 缓存池
`Objects/stringobject.c`

```c
static PyStringObject *characters[UCHAR_MAX + 1];
static PyStringObject *nullstring;

/* This dictionary holds all interned strings.  Note that references to
   strings in this dictionary are *not* counted in the string's ob_refcnt.
   When the interned string reaches a refcnt of 0 the string deallocation
   function will delete the reference from this dictionary.

   Another way to look at this is that to say that the actual reference
   count of a string is:  s->ob_refcnt + (s->ob_sstate?2:0)
*/
static PyObject *interned;
```

## list 对象

`Objects/listobject.c`

```c
/* Empty list reuse scheme to save calls to malloc and free */
#define MAXFREELISTS 80
static PyListObject *free_lists[MAXFREELISTS];
static int num_free_lists = 0;
```

## dict 对象,关联式容器
