
break in for_each

```rust
    (0..4).try_for_each(|it| {
        println!("{}",it);
        if(it>1){
            return ControlFlow::Break(());
        }
        ControlFlow::Continue(())
   }); 
```
必须使用try_for_each,普通的for_each无法实现当前功能

