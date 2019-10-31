# reset

如果某个Convey()在执行时，有一些强关联的部分(eg：db操作中的db连接)，
可以在Convey执行完或执行过程中，进行teardown处理。

    强关联操作
    Convey(){
      ...

      最后进行逆操作(或是teardown操作)
      Reset(func(){

      })
    }

下面是一个db测试操作：

    Convey("Top-level", t, func() {

        // setup (run before each `Convey` at this scope):
        db.Open()
        db.Initialize()

        Convey("Test a query", func() {
            db.Query()
            // TODO: assertions here
        })

        Convey("Test inserts", func() {
            db.Insert()
            // TODO: assertions here
        })

        Reset(func() {
            // This reset is run after each `Convey` at the same scope.
            db.Close()
        })

    })
