---
title: three学习
tags: [threejs]
categories: [threejs]
---

;(<GlobalThree>global).THREE = THREE

export interface GlobalThree extends NodeJS.Global {
THREE: any
}

材质的.sizeAttenuation 属性决定了材质大小是否会被相机深度衰减。（仅限透视摄像头。）默认为 true
