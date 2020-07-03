## android:allowTaskReparenting

Android 默认为每个 Activity 指定了一个 affinity，如果没有指定此属性，默认为 Application 标签指定的 `affinity`，若 Application 中亦没有指定，则默认为 app 的包名。Activity 的 `allowTaskReparenting` 属性没有指定时，会使用上级 `Application` 中指定的值作为它的值，当 Application 中亦没有指定时，默认值为 false。

当 A App 的 A Activity 指定了此属性为 true 时，若 B App 切换到前台，且 B App 的 Activity 与 A App 的 A Activity 有相同的 taskAffinity，则 A Activity 会从 A App 的 task 中移动到 B App 的 task 栈中来。

比如在邮件 App 中点击一个链接，会打开一个网页，这个网页是浏览器 App 中的页面，只不过此时作为 e-mail app 的一部分展示。此时如果将浏览器切换到前台，则此网页会从 e-mail app 中消失，来到浏览器 app 中。





## 参考文章



[Manifest](https://developer.android.com/guide/topics/manifest/activity-element#aff)

[关于Android TaskAffinity的那些事儿](https://blog.csdn.net/goodlixueyong/article/details/49620667)

