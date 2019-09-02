# Cereal

## cereal.hpp

### Class Archive

#### 在 Archive 里注册智能指针

```c++
//! Registers a shared pointer with the archive
/*! This function is used to track shared pointer targets to prevent
    unnecessary saves from taking place if multiple shared pointers
    point to the same data.

    @internal
    @param addr The address (see shared_ptr get()) pointed to by the shared pointer
    @return A key that uniquely identifies the pointer */
inline std::uint32_t registerSharedPointer( void const * addr )
{
  // Handle null pointers by just returning 0
  if(addr == 0) return 0;

  auto id = itsSharedPointerMap.find( addr );
  if( id == itsSharedPointerMap.end() )
  {
    auto ptrId = itsCurrentPointerId++;
    itsSharedPointerMap.insert( {addr, ptrId} );
    return ptrId | detail::msb_32bit; // mask MSB to be 1
  }
  else
    return id->second;
}
```
避免多个智能指针指向同一对象导致序列化多次，给同一地址的智能指针标号。

#### 在Archive里注册多态类型

```C++
//! Registers a polymorphic type name with the archive
/*! This function is used to track polymorphic types to prevent
    unnecessary saves of identifying strings used by the polymorphic
    support functionality.

    @internal
    @param name The name to associate with a polymorphic type
    @return A key that uniquely identifies the polymorphic type name */
inline std::uint32_t registerPolymorphicType( char const * name )
{
    auto id = itsPolymorphicTypeMap.find( name );
    if( id == itsPolymorphicTypeMap.end() )
    {
      auto polyId = itsCurrentPolymorphicTypeId++;
      itsPolymorphicTypeMap.insert( {name, polyId} );
      return polyId | detail::msb_32bit; // mask MSB to be 1
    }
    else
      return id->second;
}
```

将多态类型的名字与多态id进行注册和绑定

#### 序列化非虚继承的基类函数，一个Wrapper

```c++
//! Serialization of a base_class wrapper
/*! \sa base_class */
template <class T> inline
ArchiveType & processImpl(base_class<T> const & b)
{
  self->processImpl( *b.base_ptr );
  return *self;
}

//! Common base type for base class casting
struct BaseCastBase {};

template<class Base>
struct base_class : private traits::detail::BaseCastBase
{
  template<class Derived>
  base_class(Derived const * derived) :
  base_ptr(const_cast<Base*>(static_cast<Base const *>(derived)))
  {
    static_assert( std::is_base_of<Base, Derived>::value, "Can only use base_class on a valid base class" );
    base_class_detail::RegisterPolymorphicBaseClass<Base, Derived>::bind();
  }

  Base * base_ptr;
};

//! Used to register polymorphic relations. Polymorphic version
/*! @internal */
template <class Base, class Derived>
struct RegisterPolymorphicBaseClass<Base, Derived, true>
{
  static void bind()
  { detail::RegisterPolymorphicCaster<Base, Derived>::bind(); }
};

//! Registers a polymorphic casting relation between a Base and Derived type
/*! Registering a relation allows cereal to properly cast between the two types
    given runtime type information and void pointers.

    Registration happens automatically via cereal::base_class and        cereal::virtual_base_class instantiations. For cases where neither is called, see the CEREAL_REGISTER_POLYMORPHIC_RELATION macro */
template <class Base, class Derived>
struct RegisterPolymorphicCaster
{
  static PolymorphicCaster const * bind( std::true_type /* is_polymorphic<Base> */)
  {
    return &StaticObject<PolymorphicVirtualCaster<Base, Derived>>::getInstance();
  }

  static PolymorphicCaster const * bind( std::false_type /* is_polymorphic<Base> */ )
  { return nullptr; }

  //! Performs registration (binding) between Base and Derived
  /*! If the type is not polymorphic, nothing will happen */
  static PolymorphicCaster const * bind()
  { return bind( typename std::is_polymorphic<Base>::type() ); }
};
```

负责自动创建静态对象，关联基类对象和派生类对象的关系。每一对基类、派生类组合都有一个静态对象。

```c++
//! Base type for polymorphic void casting
/*! Contains functions for casting between registered base and derived types.

    This is necessary so that cereal can properly cast between polymorphic types
    even though void pointers are used, which normally have no type information.
    Runtime type information is used instead to index a compile-time made mapping
    that can perform the proper cast. In the case of multiple levels of inheritance,
    cereal will attempt to find the shortest path by using registered relationships to
    perform the cast.

    This class will be allocated as a StaticObject and only referenced by pointer,
    allowing a templated derived version of it to define strongly typed functions
    that cast between registered base and derived types. */
struct PolymorphicCaster
{
  PolymorphicCaster() = default;
  PolymorphicCaster( const PolymorphicCaster & ) = default;
  PolymorphicCaster & operator=( const PolymorphicCaster & ) = default;
  PolymorphicCaster( PolymorphicCaster && ) CEREAL_NOEXCEPT {}
  PolymorphicCaster & operator=( PolymorphicCaster && ) CEREAL_NOEXCEPT { return *this; }
  virtual ~PolymorphicCaster() CEREAL_NOEXCEPT = default;

  //! Downcasts to the proper derived type
  virtual void const * downcast( void const * const ptr ) const = 0;
  //! Upcast to proper base type
  virtual void * upcast( void * const ptr ) const = 0;
  //! Upcast to proper base type, shared_ptr version
  virtual std::shared_ptr<void> upcast( std::shared_ptr<void> const & ptr ) const = 0;
};
```

负责在注册的基类和派生类型之间的转换。为什么需要在多层继承关系中找到最短路径？

