# Support for iCloud KV Storage

## Motivation

`NSUbiquitousKeyValueStore` enables developers to seamlessly store and sync app preferences, configuration, and state data across all devices linked to a user's iCloud account using lightweight key-value storage. It would be beneficial to add support for this feature to enable swift-sharing, addressing the limitations of `@AppStorage` in SwiftUI.

## Implementation

The interface for `NSUbiquitousKeyValueStore` is similar to `UserDefaults`:

```swift
@Shared(.iCloudKV("isOn")) var isOn = true
```

Any changes made to this value will be synchronized to iCloud key-value storage. Further, any change made to the "isOn" key in iCloud will be immediately played back to the `@Shared` value, as demonstrated in this test:

```swift
@Test
func externalWrite() {
  NSUbiquitousKeyValueStore.default.removeObject(forKey: "isOn")

  @Shared(.iCloudKV("isOn")) var isOn = true
  #expect(isOn == true)
  NSUbiquitousKeyValueStore.default.set(false, forKey: "isOn")
  NSUbiquitousKeyValueStore.default.synchronize()
  #expect(isOn == false)
}
```

The implementation of the iCloudKV strategy will be similar to `AppStorageKey.swift`, but it will replace `UserDefaults` with `NSUbiquitousKeyValueStore`.

## Limitations

- NSUbiquitousKeyValueStore has a size limit of 1MB total for all key-value pairs
- Key length is limited to 64 bytes
- The developer needs to enable the iCloud capability.

We will address the above limitations by warning the developer with `IssueReporting`.
