## Q1： 我们在数据库操作的时候，比如 dao 层中当遇到一个 sql.ErrNoRows 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？

- 应该Wrap这个error，抛出给上层。由上层决定如何处理。

```go
// dao层
var ErrNotFound = error.New("rows not found")

func GetUserInfo() (user User, err error) {
    if err == sql.ErrNoRows {
			return nil, errors.Wrap(ErrNotFound, "user info not exits")
		}
}

// 业务层
func UserLogic() {
    userInfo,err := dao.GetUserInfo()
		if err != nil {
        if errors.Is(dao.ErrNotFound) {
				// 对客户端返回对饮的错误提示信息。
            return XXX
				} else {
					...
				}
		}
}

```

