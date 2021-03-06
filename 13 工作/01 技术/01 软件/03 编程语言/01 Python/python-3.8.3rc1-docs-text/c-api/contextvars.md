上下文变量对象
**************

注解:

  在 3.7.1 版更改: 在 Python 3.7.1 中，所有上下文变量 C API 的签名被
  **更改** 为使用 "PyObject" 指针而不是 "PyContext", "PyContextVar" 以
  及 "PyContextToken"，例如:

     // in 3.7.0:
     PyContext *PyContext_New(void);

     // in 3.7.1+:
     PyObject *PyContext_New(void);

  详情请参阅:issue: ' 34762 '。

3.7 新版功能.

This section details the public C API for the "contextvars" module.

PyContext

   The C structure used to represent a "contextvars.Context" object.

PyContextVar

   The C structure used to represent a "contextvars.ContextVar"
   object.

PyContextToken

   The C structure used to represent a "contextvars.Token" object.

PyTypeObject PyContext_Type

   The type object representing the *context* type.

PyTypeObject PyContextVar_Type

   The type object representing the *context variable* type.

PyTypeObject PyContextToken_Type

   The type object representing the *context variable token* type.

类型检查宏：

int PyContext_CheckExact(PyObject *o)

   Return true if *o* is of type "PyContext_Type". *o* must not be
   "NULL".  This function always succeeds.

int PyContextVar_CheckExact(PyObject *o)

   Return true if *o* is of type "PyContextVar_Type". *o* must not be
   "NULL".  This function always succeeds.

int PyContextToken_CheckExact(PyObject *o)

   Return true if *o* is of type "PyContextToken_Type". *o* must not
   be "NULL".  This function always succeeds.

Context object management functions:

PyObject *PyContext_New(void)
    *Return value: New reference.*

   Create a new empty context object.  Returns "NULL" if an error has
   occurred.

PyObject *PyContext_Copy(PyObject *ctx)
    *Return value: New reference.*

   Create a shallow copy of the passed *ctx* context object. Returns
   "NULL" if an error has occurred.

PyObject *PyContext_CopyCurrent(void)
    *Return value: New reference.*

   Create a shallow copy of the current thread context. Returns "NULL"
   if an error has occurred.

int PyContext_Enter(PyObject *ctx)

   Set *ctx* as the current context for the current thread. Returns
   "0" on success, and "-1" on error.

int PyContext_Exit(PyObject *ctx)

   Deactivate the *ctx* context and restore the previous context as
   the current context for the current thread.  Returns "0" on
   success, and "-1" on error.

int PyContext_ClearFreeList()

   Clear the context variable free list. Return the total number of
   freed items.  This function always succeeds.

Context variable functions:

PyObject *PyContextVar_New(const char *name, PyObject *def)
    *Return value: New reference.*

   创建一个新的' ' ContextVar' '对象。形参*name*用于自我检查和调试目的
   。可选形参*def*为上下文变量指定默认值。如果发生错误，这个函数返回'
   ' NULL ' '。

int PyContextVar_Get(PyObject *var, PyObject *default_value, PyObject **value)

   获取上下文变量的值。如果在查找过程中发生错误，返回' ' -1 ' '，如果
   没有发生错误，无论是否找到值，都返回' ' 0 ' '，

   如果找到上下文变量，*value* 将是指向它的指针。 如果上下文变量 *没有
   * 找到，*value* 将指向：

   * *default_value*，如果非``NULL``;

   * *var* 的默认值，如果不是 "NULL"；

   * "NULL"

   如果找到该值，函数将创建对它的新引用。

PyObject *PyContextVar_Set(PyObject *var, PyObject *value)
    *Return value: New reference.*

   在当前上下文中将 *var* 的值设为 *value*。 返回指向 "PyObject" 对象
   的指针，如果发生错误则返回 "NULL"。

int PyContextVar_Reset(PyObject *var, PyObject *token)

   将上下文变量 *var* 的状态重置为它在返回 *token* 的
   "PyContextVar_Set()" 被调用之前的状态。 此函数成功时返回 "0"，出错
   时返回 "-1"。
