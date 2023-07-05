---
layout: post
title: Apple 鍵盤的Fn/Globe key
date: 2023-07-04 22:52:06
categories:
  - Accessories
tags:
  - keyboard
---

前陣子買了Keychron K3 Pro 在家裡使用，結果遇到F3, F4 失效了。研究了一下才發現是karabiner 的原因，但我太依賴karabiner 了無法不用，只好想其他的解決辦法。研究的過程發現Apple 鍵盤或macbook 上的`Fn` 跟一般鍵盤的`Fn` 好像不太一樣，這邊紀錄一下。

# 一般Fn key

- 不會送出keycode。
- 通常是切換鍵盤的layer，達到送出不同的keycode。
- 功能燒在鍵盤韌體上。

# Apple Fn/Globe🌐 key (`apple_fn`)

- 是一個有keycode 的按鍵。 `{"apple_vendor_top_case_key_code": "keyboard_fn"}`
- 不是切換layer，比較像是control 這種控制鍵 modifier。
- 只有特定VID/PID 的鍵盤（= 原廠鍵盤）送出的`apple_fn`，macOS 才認可（不愧是Apple）。

# Keychron K3 Pro

- win/mac mode 其實只是切換不同layer。mac (0), mac fn (1), win (2), win fn (3)，只有四層。
- mission control (F3) 與launchpad (F4) 其實沒有送出keycode，是用某種方式模擬？或是Event Viewer 認不得？
- 因為沒有keycode，在經過karabiner 攔截後就失效了，想改都沒的改。
- 用VIA 修改：F3 → `C(KC_UP)`，F4 → `HYPR(KC_SPC)`，模擬組合鍵 ([ref](https://docs.qmk.fm/#/feature_advanced_keycodes?id=modifier-keys))。需額外修改開啟launchpad 的熱鍵為 `cmd+opt+ctrl+shift+space`。
- K7 的fn2 好像可以用apple_fn？

# Niz mini84

- mac mode 下有兩個Fn key，Fn +  fn。
- win/mac mode 連VID/PID 都不一樣。
- mac mode 用了某個Apple 原廠鍵盤的VID/PID 0x05ac/0x0220 ([link](https://devicehunt.com/view/type/usb/vendor/05AC/device/0220)，不怕被吉嗎？)，所以可以送出`apple_fn`。
- win mode 下VID/PID 是 0x0438/0x5235，是AMD…? ([link](https://devicehunt.com/search/type/usb/vendor/0438/device/any))

# Karabiner Elements

- 用虛擬鍵盤攔截實體鍵盤的keycode，修改後再送給OS。
- 如果有經過karabiner，則VID/PID 錯誤的`apple_fn` keycode 也可以正常觸發 ([ref](https://github.com/qmk/qmk_firmware/pull/20643#issuecomment-1529263497))。

# QMK

- 2023.02，新增了mission control (KC_MCTL) 與 launchpad (KC_LPAD) 的keycode ([ref](https://github.com/qmk/qmk_firmware/blob/master/data/constants/keycodes/keycodes_0.0.2_basic.hjson))。
- 因為`apple_fn` 限制VID/PID，所以QMK 無法使用這個keycode ([ref](https://github.com/qmk/qmk_firmware/pull/20643))。
- 但還是有辦法改，參考 [ref](https://gist.github.com/fauxpark/010dcf5d6377c3a71ac98ce37414c6c4)。

# Reference

- https://sspai.com/post/79608
