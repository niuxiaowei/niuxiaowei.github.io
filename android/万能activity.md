##你是如何把Activity写的如此“万能”的

##前言
------
自己android开发也有些年头了，每每回想起作为初学者的时候自己写的代码，自己会有种喷自己的冲动，代码写的太渣了。因此想着自己要总结下以前代码中的不合理的地方，希望能给初学者一些帮助。我希望这是一个系列的文章。
##本节内容
------
一个“万能”的Activity是什么样子，“万能”的Activity有哪些不好的地方
##开始编写“万能”的Activity
-----
作为了一个初学者，有可能会有好多的朋友把Activity写的很“万能”，当然没有更好。那我就以一个登陆模块为例子，来说说一个“万能”的Activity是怎么产生的，以下代码多为伪代码。

编写activity_login.xml文件，这里就简单写下伪代码：

      一个输入用户账号的 EditText
      一个输入密码的   EditText
      一个登陆按钮 Button

编写LoginActivity，LoginActivity包含初始化activity_login.xml中views的功能，还包含给登陆按钮加监听器的功能，下面看下关键的代码片段

       public LoginActivity extends Activity{
              private EditText  mUserNameView, mPasswordView;
              private Button mLoginView;
              //该方法会被onCreate方法调用
              public void initViews(){
                     .......
                      各种findViewById.....代码

                      //给登陆按钮加监听器
                      mLoginView.OnClickListener(new View.OnClickListener() {
                            @Override
                             public void onClick(View v) {    
                            }
                      });
              }
       }

现在的LoginActivity中的代码是不是还很清爽，干净。它只做与UI相关的工作。

**具有验证功能的LoginActivity**
那接着要在登陆按钮的监听器方法实现验证账号，密码是否有效的功能，继续接着完善代码

        mLoginView.OnClickListener(new View.OnClickListener() {
              @Override
              public void onClick(View v) {

                    String userName = mUserNameView.getText();
                    String password = mPasswordView.getText();
                     //验证用户输入的用户名，密码是否合法
                    if(!validate(userName) || !validate(password)){
                            告诉用户输入的用户名或密码不合法，并做一些其他的工作
                     }
              }
         });

         //验证给定的字符串是否合法，true 合法，false 不合法
         private  boolean validate(String str){

         }

  现在的LoginActivity已经有业务代码出现了，我们该实现登陆功能了。

**具有登陆功能的LoginActivity**
在监听器中增加登陆功能

        mLoginView.OnClickListener(new View.OnClickListener() {
              @Override
              public void onClick(View v) {
                    验证用户输入的用户名，密码是否合法代码...
                    if(都合法）｛
                          //开始登陆
                          login(userName,password);
                    ｝
              }
         });
          //登陆方法,用伪代码来写下网络请求
         private void login(String userName,String password){

                HttpClient.getInstance().login(userName,password,
                       new ResponseListener(){

                               public void failed(Failed failed){
                                      做失败相关的处理工作，比如给用户提示
                                      把密码输入框清空，还比如登陆次数限制等
                               }

                               public void success(Response response){
                                      对数据进行解析，跳转到app主页或其他界面
                               }
                       });
         }

**具有数据解析，存储功能的LoginActivity**
当我把登陆功能做好后，这时候产品又提了新需求，需要在用户登陆成功后，把用户相关的数据返回，并保存。那没辙只能按要求来做，但是幸运的是需求很简单，只需要修改下登陆成功方法就ok。

          public void success(Response response){
              做成功相关的处理工作
              //暂且把用户信息的类叫做UserInfo,从json中解析数据，假设response.getContent()存在
              String jsonContent = response.getContent();
              JsonObject jsonObject = new JsonObject(jsonContent);
              UserInfo userInfo = new UserInfo();
              userInfo.name = jsonObject.optString("name");
              userInfo.userId = jsonObject.optString("userId");
              其他字段的解析......
              //保存userInfo信息到数据表中,假设userDatabase已经存在
              userDatabase.save(userInfo);

              跳到app的主页
          }

到此，登陆功能开发完毕，LoginActivity最后的代码是这样

         public LoginActivity extends Activity{
              private EditText  mUserNameView, mPasswordView;
              private Button mLoginView;

              public void initViews(){
                     .......
                      各种findViewById.....代码

                      //给登陆按钮加监听器
                      mLoginView.OnClickListener(new View.OnClickListener() {
                            @Override
                             public void onClick(View v) {  

                                  String userName = mUserNameView.getText();
                                  String password = mPasswordView.getText();
                                  //验证用户输入的密码是否合法
                                  if(!validate(userName) || !validate(password)){
                                      告诉用户输入的用户名或密码不合法
                                  }  else{
                                      //开始登陆
                                      login(userName,password);
                                  }
                            }
                      });
              }

               //登陆方法,用伪代码来写下网络请求
               private void login(String userName,String password){

                    HttpClient.getInstance().login(userName,password,
                       new ResponseListener(){

                               public void failed(Failed failed){
                                      做失败相关的处理工作，比如给用户提示
                                      把密码输入框清空，还比如登陆次数限制等
                               }

                               public void success(Response response){
                                       做成功相关的处理工作
                                      //暂且把用户信息的类叫做UserInfo,从json中解析数据，假设response.getContent()存在
                                      String jsonContent = response.getContent();
                                      JsonObject jsonObject = new JsonObject(jsonContent);
                                      UserInfo userInfo = new UserInfo();
                                      userInfo.name = jsonObject.optString("name");
                                      userInfo.userId = jsonObject.optString("userId");
                                      其他字段的解析......
                                      //保存userInfo信息到数据表中,假设userDatabase已经存在
                                      userDatabase.save(userInfo);

                                      跳到app的主页
                               }
                     });
               }

                //验证给定的字符串是否合法，true 合法，false 不合法
               private  boolean validate(String str){

               }
       }

到此登陆模块功能已经开发完成，LoginActivity由开始的清爽，干净的只关心U的代码，变成了现在的“万能”Activity，LoginActivity都做了哪些工作：
  - 处理UI
  - 处理用户名，密码合法性验证业务
  - 处理数据解析，数据存储

当然你会说现在的LoginActivity的代码也就将近100行，修改起来还是很容易的，但是随着业务的增长，谁能保证LoginActivity代码不会越来越多，包含的业务代码不会更多。LoginActivity只是"万能"Activity的一个缩影而已。

万能是好事情，但是得看万能是通过什么方式实现的，假如是通过把好多小功能分散到各自的类中，然后万能类通过把这些小类组合在一起的万能是真材实料的万能。若是通过把好多小功能全部放在一个类中使得这个类变的“万能”了，这种“万能”是不好的。

**“万能”Activity缺点**
我总结下“万能”Activity缺点
  - 引起"万能"Activity的变化因素很多，所以动不动就得修改它，没办法使得由于修改引入的bug的机率降到最低。
  - 业务功能没法复用
  - 这样的Activity可维护性，可阅读性都不好，耦合很高。

所以为了以后长远考虑，遇到“万能”Activity一定要大胆的重构，虽然当前重构需要花很多时间，但是以后是一劳永逸的，因此让“万能”Activity不在万能，让**Activity只做与UI相关的工作**就足够了。

##总结
关于“万能”Activity的事情就聊到这，请看[Android:“万能”Activity重构篇](http://www.jianshu.com/p/559f85a42f23)
代码清爽，干净了，阅读起来精神倍儿爽，一口气阅读100行不成问题！

本人微信：704451290

![本人公众账号](http://upload-images.jianshu.io/upload_images/1504173-9cf9f1ab13509e8c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
