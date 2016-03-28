#前言

今天做到这个么小需求，需要添加N条手机号到通讯录，同样也要能清空通讯录。在完成后， 将这两段代码片段记录下来，供大家参考！

#添加到通讯录

下面是添加到通讯录的一段代码片段，这里只是手机号作为firstName，功能很单一，具体要添加多个值需要自己去添加！

```
CFErrorRef error = NULL;
  
//创建一个通讯录操作对象
ABAddressBookRef addressBook = ABAddressBookCreateWithOptions(NULL, &error);
  
ABAddressBookRequestAccessWithCompletion(addressBook, ^(bool granted, CFErrorRef error) {
if (granted && !error) {
  for (NSUInteger i = 0; i < self.textField.text.integerValue && i < count; ++i) {
    @autoreleasepool {
      // 创建一条新的联系人纪录
      ABRecordRef newRecord = ABPersonCreate();
      
      // 为新联系人记录添加属性值
      ABRecordSetValue(newRecord,
                       kABPersonFirstNameProperty,
                       (__bridge CFTypeRef)self.phoneArray[i],
                       &error);
      
      //创建一个多值属性
      ABMutableMultiValueRef multi = ABMultiValueCreateMutable(kABMultiStringPropertyType);
      ABMultiValueAddValueAndLabel(multi,
                                   (__bridge CFTypeRef)self.phoneArray[i],
                                   kABPersonPhoneMobileLabel, NULL);
      
      //将多值属性添加到记录
      ABRecordSetValue(newRecord, kABPersonPhoneProperty, multi, &error);
      //添加记录到通讯录操作对象
      ABAddressBookAddRecord(addressBook, newRecord, &error);
      
      CFRelease(newRecord);
      CFRelease(multi);
    }
  }
  
  //保存通讯录操作对象
  ABAddressBookSave(addressBook, &error);
  CFRelease(addressBook);
}
```

#清空通讯录

下面是一段清空通讯录的代码片段，一定要小心哦，清空前一定要慎重！

```
CFErrorRef error = NULL;
  
//创建一个通讯录操作对象
ABAddressBookRef addressBook = ABAddressBookCreateWithOptions(NULL, &error);
  
ABAddressBookRequestAccessWithCompletion(addressBook, ^(bool granted, CFErrorRef error) {
	if (granted && !error) {
	  CFArrayRef personArray = ABAddressBookCopyArrayOfAllPeople(addressBook);
	  CFIndex personCount = ABAddressBookGetPersonCount(addressBook);
	  
	  if (personCount <= 0) {
	    dispatch_async(dispatch_get_main_queue(), ^{
	      [SVProgressHUD showSuccessWithStatus:@"清空通讯录成功"];
	    });
	    return;
	  }
	  
	  for (int i = 0; i < personCount; i++) {
	    ABRecordRef ref = CFArrayGetValueAtIndex(personArray, i);
	    // 删除联系人
	    ABAddressBookRemoveRecord(addressBook, ref, nil);
	  }
	  
	  // 保存通讯录操作对象
	  ABAddressBookSave(addressBook, &error);
	  CFRelease(addressBook);
	  
	  dispatch_async(dispatch_get_main_queue(), ^{
	    if (!error) {
	      [SVProgressHUD showSuccessWithStatus:@"清空通讯录成功"];
	    } else {
	      [SVProgressHUD showErrorWithStatus:@"清空通讯录失败"];
	    }
	  });
	}
});
```

#最后

iOS9.0以后ABAddressBook这个framework被废弃了，推荐的是CNContact这个类来处理。不过我们都需要兼容9.0以下版本，所以现在不用管它。