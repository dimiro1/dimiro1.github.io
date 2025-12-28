+++
title = 'How we made buttons before CSS3'
date = 2025-12-28
description = ''
tags = ['CSS', 'HTML']
draft = true
+++

Content coming soon.

## The Glossy Aqua Button

{{< html-demo title="Aqua Button" >}}
<style>
  .aqua-btn {
    font-family: Arial, sans-serif;
    font-size: 14px;
    font-weight: bold;
    color: #fff;
    text-shadow: 0 -1px 1px rgba(0,0,0,0.4);
    padding: 10px 24px;
    border: 1px solid #2d6da3;
    border-radius: 8px;
    background: linear-gradient(to bottom, #7dc5ee 0%, #4ca1d9 50%, #2d8bc9 51%, #1a6ba0 100%);
    box-shadow: 0 2px 4px rgba(0,0,0,0.3), inset 0 1px 0 rgba(255,255,255,0.4);
    cursor: pointer;
  }
  .aqua-btn:hover {
    background: linear-gradient(to bottom, #8fd0f5 0%, #5eb0e8 50%, #3e9bd8 51%, #2a7bb0 100%);
  }
  .aqua-btn:active {
    box-shadow: inset 0 2px 4px rgba(0,0,0,0.3);
  }
</style>
<button class="aqua-btn">Download Now</button>
{{< /html-demo >}}

## The Web 2.0 Gradient Button

{{< html-demo title="Web 2.0 Button" >}}
<style>
  .web2-btn {
    font-family: "Lucida Grande", Tahoma, sans-serif;
    font-size: 13px;
    font-weight: bold;
    color: #333;
    text-shadow: 0 1px 0 rgba(255,255,255,0.8);
    padding: 8px 20px;
    border: 1px solid #999;
    border-radius: 5px;
    background: linear-gradient(to bottom, #fff 0%, #e6e6e6 50%, #ccc 51%, #f5f5f5 100%);
    box-shadow: 0 1px 3px rgba(0,0,0,0.2);
    cursor: pointer;
  }
  .web2-btn:hover {
    background: linear-gradient(to bottom, #fff 0%, #f0f0f0 50%, #ddd 51%, #fff 100%);
  }
</style>
<button class="web2-btn">Sign Up Free</button>
{{< /html-demo >}}

## The Candy Button

{{< html-demo title="Candy Button" modals="true" >}}
<style>
  .candy-btn {
    font-family: Georgia, serif;
    font-size: 16px;
    font-weight: bold;
    color: #fff;
    text-shadow: 0 -1px 1px rgba(0,0,0,0.5);
    padding: 12px 30px;
    border: none;
    border-radius: 20px;
    background: linear-gradient(to bottom, #ff7f50 0%, #ff6347 50%, #dc143c 100%);
    box-shadow: 0 4px 0 #8b0000, 0 6px 10px rgba(0,0,0,0.3);
    cursor: pointer;
    position: relative;
    top: 0;
    transition: all 0.1s;
  }
  .candy-btn:active {
    top: 4px;
    box-shadow: 0 0 0 #8b0000, 0 2px 5px rgba(0,0,0,0.3);
  }
</style>
<button class="candy-btn" onclick="alert('Loading game... just kidding!')">Play Now!</button>
{{< /html-demo >}}
