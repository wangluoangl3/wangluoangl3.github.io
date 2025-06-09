---
layout: post
title: "fix the bug of sb3_highway_dqn"
date: 2024-06-08
catalog: true
tags:
    - Reinforcement Learning
---


# 1. bug that occured
<img src="/img-post/reinforcement_learning_error.png" alt="RecordVideo object has no attribute video_recorder">


# 2. fix the bug
we fix the file "/usr/local/lib/python3.11/dist-packages/highway_env/envs/common/abstract.py" line 337 and line 338 by removing self._record_video_wrapper.video_recorder and change "self._record_video_wrapper.video_recorder.capture_frame into "self._record_video_wrapper.video_recorder._capture_frame()". the fixed code like this below:
<img src="/img-post/reinforcement_learning_error_fixed.png" alt="fixed error">


