# Logging In (JSON)

```js
export login = catchAsync(async (req, res, next) => {
  const { email, password } = req.body;
  if (!email || !password) {
    return next(new AppError('Please provide an email and password', 400));
  }

  const user = await User.findOne({ email }).select('+password');

  if (!user || !(await user.matchPassword(password, user.password)))
    return next(new AppError('Incorrect email or password', 401));

  sendJWT(user, 200, res);
})
```