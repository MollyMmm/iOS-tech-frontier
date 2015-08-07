# Swift视频录制开发指南 
===

> * 原文链接 : [Take Video Tutorial in iOS8 with Swift](http://www.ioscreator.com/tutorials/take-video-tutorial-ios8-swift)
* 原文作者 : [ioscreator](www.ioscreator.com)
* [译文出自 :  开发技术前线 www.devtf.cn](http://www.devtf.cn)
* 译者 : [Mr.Simple](https://github.com/bboyfeiyu) 
* 校对者: [StormXX](https://github.com/StormXX) 

Apple提供了一个UIImagePickerController用户界面类来让用户使用内置的摄像头来拍摄视频。我的开发环境是iOS 8.4和xcode 6.4，在这篇文章中我会教大家获取已经存储在Photo Library的视频。

首先打开Xcode，然后创建一个名为IOS8SwiftTakeVideoPlayerTutorial的应用，Organization名和Organization标识符使用你自己的即可。选择Swift作为开发语言，并且该应用只支持Iphone。

![](http://static1.squarespace.com/static/52428a0ae4b0c4a5c2a2cede/t/559ce50ee4b0560c42b45099/1436345615076/)

到Storyboard页面，从Object Library拖两个按钮到主视图上，给这两个按钮设置title,分别为"Take Video" 和 "View Library"，Storyboard此时应该如下图所示: 

![](http://static1.squarespace.com/static/52428a0ae4b0c4a5c2a2cede/t/559ce595e4b0cebfa4eee243/1436345750826/)

按住Ctrl键，然后选中这两个按钮，点击在Storyboard右下方的“Resolve Auto Layout Issues”按钮，然后选择“Add Missing Constraints”。如下图所示 : 

![](http://static1.squarespace.com/static/52428a0ae4b0c4a5c2a2cede/t/559ce604e4b04c3e89ff0e2e/1436345862940/)

切换到Assistant Editor界面，并且确保ViewController.swift是可见的。按住Ctrl并从Take Video按钮区域拖出以创建一个Action.

![](http://static1.squarespace.com/static/52428a0ae4b0c4a5c2a2cede/t/559ce655e4b051cc74e74388/1436345942333/)

同样的，按住按住Ctrl并从View Library按钮区域拖出以创建一个Action。

![](http://static1.squarespace.com/static/52428a0ae4b0c4a5c2a2cede/t/559ce67fe4b001bba4e7e8e9/1436345984380/)

切换到ViewController.swfit文件，在文件的头部添加如下代码。

```
import MobileCoreServices
import AssetsLibrary
```

修改ViewController类的定义如下 : 

`class ViewController: UIViewController, UINavigationControllerDelegate, UIImagePickerControllerDelegate {
`

这些delegates使得ViewController能够处理UIImagePickerController的代理事件，takeVideo的实现如下 : 

```
@IBAction func takeVideo(sender: AnyObject) {
    // 1 Check if project runs on a device with camera available
    if UIImagePickerController.isSourceTypeAvailable(.Camera) {
            
        // 2 Present UIImagePickerController to take video
        controller.sourceType = .Camera
        controller.mediaTypes = [kUTTypeMovie as! String]
        controller.delegate = self
        controller.videoMaximumDuration = 10.0
            
        presentViewController(controller, animated: true, completion: nil)
     }
     else {
        println("Camera is not available")
     }
}
```

1. isSourceTypeAvailable函数是检测设备的Camera是否可用；
2. ImagePickerController用于显示Sourcetype为Camera、mediaType为Movie的视频，视频的最大长度为10秒。

viewLibrary的实现 : 

```
@IBAction func viewLibrary(sender: AnyObject) {
            // Display Photo Library
            controller.sourceType = UIImagePickerControllerSourceType.PhotoLibrary
            controller.mediaTypes = [kUTTypeMovie as! String]
            controller.delegate = self
            
            presentViewController(controller, animated: true, completion: nil)
    }
```

在调用上述代码之后，Photo library就会被显示。如果mediaType没有设置为Movie类型，视频文件就不会被显示出来。下一步就是实现UIImagePickerControllerDelegate协议的方法 : 

```
func imagePickerController(picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [NSObject: AnyObject]) {
    // 1    
    let mediaType:AnyObject? = info[UIImagePickerControllerMediaType]
        
    if let type:AnyObject = mediaType {
        if type is String {
            let stringType = type as! String
                if stringType == kUTTypeMovie as! String {
                    let urlOfVideo = info[UIImagePickerControllerMediaURL] as? NSURL
                        if let url = urlOfVideo {
                            // 2  
                            assetsLibrary.writeVideoAtPathToSavedPhotosAlbum(url,
                                completionBlock: {(url: NSURL!, error: NSError!) in
                                        if let theError = error{
                                            println("Error saving video = \(theError)")
                                        }
                                        else {
                                            println("no errors happened")
                                        }
                                })
                        }
                } 
        }
    }
    // 3
    picker.dismissViewControllerAnimated(true, completion: nil)
}
```

当用户选择了一个视频时`imagePickerController(\_:didFinishPickingMediaWithInfo:)`函数会被调用，info参数中会包含被选择的视频URL。

1. 首先会检测info dictionary中的mediatype是否是movie类型，如果是那么则会提取这个视频的URL；
2. 调用writeVideoAtPathToSavedPhotosAlbum函数将视频存储到图片相册中；
3. 隐藏该ViewController。

`imagePickerControllerDidCancel`实现如下 : 

```
func imagePickerControllerDidCancel(picker: UIImagePickerController) {
        picker.dismissViewControllerAnimated(true, completion: nil)
}
```

当用户按下取消按钮，该界面的View Controller就会被隐藏。因为模拟器没有摄像头，因此编译并且将该项目运行到真实的设备，点击"Take Video"按钮来拍一段视频，然后点击“Use Video”。再下一步选择“View Library”，此时该Video就会显示在Library中了。

![TakeVideo-Device.png](http://static1.squarespace.com/static/52428a0ae4b0c4a5c2a2cede/t/559d107ee4b0a65ec39328be/1436356736349/TakeVideo-Device.png)

你可以在[这里](https://github.com/ioscreator/ioscreator)下载IOS8SwiftTakeVideoPlayerTutorial项目的完整代码。

