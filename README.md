# hashmap_c.c3l
hashmap.c binding for C3

# Usage
```c3
module hashmap_example;

import std::io;
import libc;
import hashmap_c;

alias hmc = module hashmap_c;

struct User
{
  ZString name;
  CInt age;
}

fn CInt user_compare(void *a,void *b,void *udata)
{
  User *ua = a;
  User *ub = b;
  return libc::strcmp(ua.name,ub.name);
}

fn CBool user_iter(void *item,void *udata)
{
  User *user = item;
  io::printfn("%s (age=%d)",user.name,user.age);
  return (CBool)1;
}

fn ulong user_hash(void *item,ulong seed0,ulong seed1)
{
  User *user = item;
  return hmc::sip(user.name,libc::strlen(user.name),seed0,seed1);
}

fn int main(String[] args)
{
  hmc::HashMap *map = hmc::new(User::size,0,0,0,
			       &user_hash,&user_compare,null,null);
  defer hmc::hashmap_free(map);
  
  hmc::hashmap_set(map,&&(User){.name = "Dale", .age = 44});
  hmc::hashmap_set(map,&&(User){.name = "Roger", .age = 68});
  hmc::hashmap_set(map,&&(User){.name = "Jane", .age = 47});

  User *user;

  io::printfn("\n-- get some users --");
  user = hmc::hashmap_get(map,&&(User){.name = "Jane"});
  io::printfn("%s age=%d",user.name,user.age);

  user = hmc::hashmap_get(map,&&(User){.name = "Roger"});
  io::printfn("%s age=%d",user.name,user.age);
  
  user = hmc::hashmap_get(map,&&(User){.name = "Dale"});
  io::printfn("%s age=%d",user.name,user.age);

  user = hmc::hashmap_get(map,&&(User){.name = "Tom"});
  io::printfn("%s",user ? "exists" : "not exists");

  io::printfn("\n-- iterate over all users (hashmap_scan) --");
  hmc::hashmap_scan(map,&user_iter,null);

  io::printfn("\n-- iterate over all users (hashmap_iter) --");
  usz iter;
  void *item;
  while(hmc::hashmap_iter(map,&iter,&item))
  {
    User *user2 = item;
    io::printfn("%s (age=%d)",user2.name,user2.age);
  }

  return 0;
}
```

