source 'https://github.com/CocoaPods/Specs.git'
# Uncomment the next line to define a global platform for your project
platform :ios, '9.0'

project 'EMSMobileSDK.xcodeproj'
target 'EMSMobileSDK' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!
  # Pods for EMSMobileSDK
  #do not use optimistic operator '~>' as this will cause minor releases to pull in, i.e. 4.5.1
  pod 'Alamofire', '4.8.2'
  pod 'SwiftLint'

  target 'EMSMobileSDKTests' do
    inherit! :search_paths
    pod 'Alamofire', '4.8.2'
  end

end
