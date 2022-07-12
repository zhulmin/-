

#### 统计（Firebase）



**用户模型**  (Dimension)

```swift
static func logDimension(_ userModel:UserModule) {
        
        //google analysis 自定义维度统计
        if let userId = userModel.userId {
            Analytics.setUserID(userId)
        }
        if let gender = userModel.sex?.rawValue {
            Analytics.setUserProperty(gender, forName: "vi_gender")
        }
        let age = ViChatCommon.getUserAge(birthday: userModel.birthday)
        if age != "0" {
            Analytics.setUserProperty(age, forName: "vi_age")
        }
        if let name = userModel.nickname {
            Analytics.setUserProperty(name, forName: "vi_nickname")
        }
}

```



**崩溃统计**

```swift
import FirebaseCrashlytics

Crashlytics.crashlytics().setUserID(userId)
```

```bash
# 脚本配置

# Type a script or drag a script file from your workspace to insert its path.

if [ "${CONFIGURATION}" = "Release" ]; then

cp "${SRCROOT}/${TARGET_NAME}/Resources/GoogleAnalysis/GoogleService-Info-product.plist" "${SRCROOT}/${TARGET_NAME}/Resources/GoogleAnalysis/GoogleService-Info.plist"

"${PODS_ROOT}/FirebaseCrashlytics/run"

fi

if [ "${CONFIGURATION}" = "Debug" ]; then

cp "${SRCROOT}/${TARGET_NAME}/Resources/GoogleAnalysis/GoogleService-Info-debug.plist" "${SRCROOT}/${TARGET_NAME}/Resources/GoogleAnalysis/GoogleService-Info.plist"

"${PODS_ROOT}/FirebaseCrashlytics/run"

fi
```



#### 广告

* 只有通过广告下载的App可以拿到关键词（不需要申请ATT权限），自然流量的关键词是拿不到的。

* 只能拿到自己投放广告时用的词(关联到用户搜索的词)，拿不到用户搜索时用的词

* 如果广告投放的关键词用的广泛匹配，拿不到用户实际搜索的词，只能归因到自己投放的词(广泛匹配)

```swift
@available(iOS 14.5, *)
    static func fetchAttributionData() {
        
        if let adAttributionToken = try? AAAttribution.attributionToken() {
            
            let request = NSMutableURLRequest(url: URL(string:"https://api-adservices.apple.com/api/v1/")!)
            request.httpMethod = "POST"
            request.setValue("text/plain", forHTTPHeaderField: "Content-Type")
            //            Data.base64EncodedString(adAttributionToken)
            //               request.httpBody = Data(adAttributionToken)
            request.httpBody = adAttributionToken.data(using: .utf8)
            
            let task = URLSession.shared.dataTask(with: request as URLRequest) { (data, _, error) in
                
                
                if let error = error {
                    print(error)
                    return
                }
                do {
                    let jsonResponse = try JSONSerialization.jsonObject(with: data!, options: .allowFragments) as! [String:Any]
                    if let campaignId = jsonResponse["campaignId"] as? Int {
                        //Send Data to APP Backend
                        
                    }
                } catch {
                    print(error)
                }
            }
            task.resume()
        } else {
            ADClient.shared().requestAttributionDetails({ (attributionDetails, error) in
                guard let attributionDetails = attributionDetails else {
                    print("Search Ads error: \(error?.localizedDescription ?? "")")
                    return
                }
                for (version, adDictionary) in attributionDetails {
                    print("Search Ads version:", version)
                    if var adAttributionInfo = adDictionary as? Dictionary<String, Any> {
                        
                        if let campaignId = adAttributionInfo["iad-campaign-id"] as? String {
                            //Send Data to APP Backend
                        }
                    }
                }
            })
            
        }
    }
```

