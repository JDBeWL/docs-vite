---
title: 学习分享
date: 2025-07-24
tags: [learing]
pinned: true
head:
  - - meta
    - name: description
      content: 学习分享
  - - meta
    - name: keywords
      content: learn learning Share Sharing
---

# 笔记

::: tip
只是在学习的时候写的东西，推荐使用搜索功能搜索想到的笔记
:::

## HTML

### 日历
![日历](/src/share/calendar.png "日历")
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8" />
    <title>日历</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }

        #calendar {
            border: 1px solid black;
            margin: 20px auto;
            padding: 5px;
            font-size: 20px;
            line-height: 20px;
            width: 300px;
        }

        #calendar #calendar-control {
            display: flex;
            justify-content: space-between;
            margin-bottom: 5px;
        }

        #calendar #calendar-table {
            border-collapse: collapse;
            padding-top: 5px;
            display: block;
        }

        #calendar-table tr td {
            border: 1px solid black;
            text-align: center;
            padding: 5px;
            width: calc(300px/7);
        }

        button {
            width: 20px;
        }
    </style>
</head>

<body>
    <div id="calendar">
        <div id="calendar-control">
            <button onclick="formatGetTime('prev')"> &lt; </button>
            <div id="calendar-date">

            </div>
            <button onclick="formatGetTime('next')"> &gt; </button>
        </div>
        <hr />
        <table id="calendar-table">
            <tr>
                <td>一</td>
                <td>二</td>
                <td>三</td>
                <td>四</td>
                <td>五</td>
                <td>六</td>
                <td>日</td>
            </tr>
        </table>
    </div>
</body>
<script>
    let calendar = document.getElementById('calendar');
    // 添加table前从Date对象中拿取必要的值
    let date = new Date();
    // 装载时间
    let year, month, day, week, firstDay, lastDay;
    function initialTime(date) {
        year = date.getFullYear();
        month = date.getMonth();
        day = date.getDate();
        week = date.getDay();
        firstDay = new Date(year, month, 1).getDay();
        lastDay = new Date(year, month + 1, 0).getDate();
        nowDate = document.getElementById('calendar-date');
        nowDate.textContent = `${year}年${month + 1}月`;
    }
    // 使用date初始化calendar
    initialTime(date);

    // 往calendar添加table
    function addCalendar(firstDay, lastDay) {
        let table = document.getElementById('calendar-table');
        let tbody = table.getElementsByTagName('tbody')[0];
        let space = firstDay === 0 ? 6 : firstDay - 1;
        let row = Math.ceil((space + lastDay) / 7);
        let col = 7;
        let insertDay = 1;
        let today = new Date();
        let nowYear = today.getFullYear();
        let nowMonth = today.getMonth();
        let nowDay = today.getDate();
        for (let i = 1; i <= row; i++) {
            let tr = document.createElement("tr");
            for (let j = 1; j <= col; j++) {
                let td = document.createElement("td");
                if (i == 1 && j <= space) {
                    tr.append(td);
                } else {
                    if (insertDay <= lastDay) {
                        td.textContent = insertDay;
                        if (year == nowYear && month == nowMonth && insertDay == nowDay) {
                            td.style.backgroundColor = 'grey';
                            td.style.color = 'red';
                        }
                        insertDay++;
                    }
                    tr.append(td);
                }
            }
            tbody.append(tr);
        }
    }

    // 初始化默认值为当前月份的日历
    addCalendar(firstDay, lastDay);

    // 日历操作
    function formatGetTime(type) {
        switch (type) {
            case 'prev':
                let prevDate = new Date(year, month - 1, 1);
                initialTime(prevDate);
                deleteCalendar();
                addCalendar(firstDay, lastDay);
                break;
            case 'next':
                let nextDate = new Date(year, month + 1, 1);
                initialTime(nextDate);
                deleteCalendar();
                addCalendar(firstDay, lastDay);
                break;
        }
    }

    // 清空日历
    function deleteCalendar() {
        let tbody = calendar.getElementsByTagName('tbody')[0];
        let rows = calendar.querySelectorAll('tr');
        for (let i = rows.length - 1; i >= 1; i--) {
            rows[i].remove();
        }
    }


</script>

</html>
```

### 更好看的日历
![更好看的日历](/src/share/betterCalendar.png "更好看的日历")
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8" />
    <title>日历</title>
    <style>
        :root {
            --primary-color: #007bff;
            --accent-color: #f8f9fa;
            --border-radius: 8px;
            --transition: all 0.2s ease;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        #calendar {
            border: 1px solid #dee2e6;
            margin: 2rem auto;
            padding: 1.5rem;
            font-size: 1.1rem;
            width: 90%;
            max-width: 500px;
            border-radius: var(--border-radius);
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
            background: #fff;
            transition: var(--transition);
        }

        #calendar:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 16px rgba(0,0,0,0.15);
        }

        #calendar-control {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 1rem;
            padding-bottom: 0.5rem;
            border-bottom: 1px solid #dee2e6;
        }

        #calendar-table {
            border-collapse: collapse;
            width: 100%;
        }

        #calendar-table thead tr {
            background-color: #f8f9fa;
        }

        #calendar-table td, 
        #calendar-table th {
            border: 1px solid #dee2e6;
            text-align: center;
            padding: 0.5rem;
            width: calc(100% / 7);
            transition: var(--transition);
        }

        #calendar-table td.today {
            background-color: var(--primary-color);
            color: white;
            border-radius: 50%;
            width: 2.5rem;
            height: 2.5rem;
            margin: 0 auto;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        button {
            width: 2.5rem;
            height: 2.5rem;
            background: var(--accent-color);
            border: 1px solid #ced4da;
            border-radius: var(--border-radius);
            cursor: pointer;
            transition: var(--transition);
            display: flex;
            align-items: center;
            justify-content: center;
            flex-shrink: 0;
        }

        button:hover {
            background: #e9ecef;
            transform: scale(1.05);
        }
    </style>

</head>

<body>
    <div id="calendar">
        <div id="calendar-control">
            <button onclick="formatGetTime('prev')"> &lt; </button>
            <div id="calendar-date">

            </div>
            <button onclick="formatGetTime('next')"> &gt; </button>
        </div>
        <table id="calendar-table">
            <thead>
                <tr>
                    <th>一</th>
                    <th>二</th>
                    <th>三</th>
                    <th>四</th>
                    <th>五</th>
                    <th>六</th>
                    <th>日</th>
                </tr>
            </thead>
            <tbody>
            </tbody>
        </table>
    </div>
</body>
<script>
    let calendar = document.getElementById('calendar');
    // 添加table前从Date对象中拿取必要的值
    let date = new Date();
    // 装载时间
    let year, month, day, week, firstDay, lastDay;
    function initialTime(date) {
        year = date.getFullYear();
        month = date.getMonth();
        day = date.getDate();
        week = date.getDay();
        firstDay = new Date(year, month, 1).getDay();
        lastDay = new Date(year, month + 1, 0).getDate();
        nowDate = document.getElementById('calendar-date');
        nowDate.textContent = `${year}年${month + 1}月`;
    }
    // 使用date初始化calendar
    initialTime(date);

    // 往calendar添加table
    function addCalendar(firstDay, lastDay) {
        let table = document.getElementById('calendar-table');
        let tbody = table.getElementsByTagName('tbody')[0];
        let space = firstDay === 0 ? 6 : firstDay - 1;
        let row = Math.ceil((space + lastDay) / 7);
        let col = 7;
        let insertDay = 1;
        let today = new Date();
        let nowYear = today.getFullYear();
        let nowMonth = today.getMonth();
        let nowDay = today.getDate();
        for (let i = 1; i <= row; i++) {
            let tr = document.createElement("tr");
            for (let j = 1; j <= col; j++) {
                let td = document.createElement("td");
                if (i == 1 && j <= space) {
                    tr.append(td);
                } else {
                    if (insertDay <= lastDay) {
                        td.textContent = insertDay;
                        if (year == nowYear && month == nowMonth && insertDay == nowDay) {
                            td.style.backgroundColor = 'grey';
                            td.style.color = 'red';
                        }
                        insertDay++;
                    }
                    tr.append(td);
                }
            }
            tbody.append(tr);
        }
    }

    // 初始化默认值为当前月份的日历
    addCalendar(firstDay, lastDay);

    // 日历操作
    function formatGetTime(type) {
        switch (type) {
            case 'prev':
                let prevDate = new Date(year, month - 1, 1);
                initialTime(prevDate);
                deleteCalendar();
                addCalendar(firstDay, lastDay);
                break;
            case 'next':
                let nextDate = new Date(year, month + 1, 1);
                initialTime(nextDate);
                deleteCalendar();
                addCalendar(firstDay, lastDay);
                break;
        }
    }

    // 清空日历
    function deleteCalendar() {
        let tbody = calendar.getElementsByTagName('tbody')[0];
        let rows = calendar.querySelectorAll('tr');
        for (let i = rows.length - 1; i >= 1; i--) {
            rows[i].remove();
        }
    }


</script>

</html>
```

### 倒计时
![倒计时](/src/share/countdown.png "倒计时")
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8" />
    <title>倒计时</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }

        #countdownTimer {
            font-size: 5vw;
            margin: 40vh auto;
            text-align: center;
        }
    </style>
</head>

<body>
    <div id="countdownTimer">

    </div>
</body>
<script>
    let date = new Date();
    let year, month, day, week;
    let text = document.getElementById('countdownTimer');
    // 装载时间
    function initialTime(date) {
        year = date.getFullYear();
        month = date.getMonth();
        day = date.getDate();
        week = date.getDay();
    };
    initialTime(date);
    // 国庆节时间
    let targetDate = new Date(year, 9, 1);

    function countdown() {
        let now = new Date();
        // 如果今年没法过国庆节就只有明年再过，就像今年的中秋不是完整的中秋
        if (now > targetDate) {
            targetDate = new Date(targetYear + 1, 9, 1);
        }
        let years = targetDate.getFullYear() - now.getFullYear();
        let months = targetDate.getMonth() - now.getMonth();
        let days = targetDate.getDate() - now.getDate();
        // 处理日期差值
        if (days < 0) {
            months--;
            let lastMonth = new Date(targetDate.getFullYear(), targetDate.getMonth(), 0);
            days += lastMonth.getDate();
        }
        if (months < 0) {
            years--;
            months += 12;
        }
        // 时分秒计算
        let timeDiff = Math.floor((targetDate - now) % (1000 * 60 * 60 * 24) / 1000);
        let hoursDiff = Math.floor(timeDiff / 3600);
        let minutesDiff = Math.floor((timeDiff - hoursDiff * 3600) / 60);
        let secondsDiff = Math.floor(timeDiff - hoursDiff * 3600 - minutesDiff * 60);
        let totalMonths = years * 12 + months;
        // 输出格式调整，如果有月份则输出月份，如果没有则不输出月份
        if (totalMonths > 0) {
            text.textContent = `距离国庆节还剩${totalMonths}月${days}天${hoursDiff}时${minutesDiff}分${secondsDiff}秒`;
        } else {
            text.textContent = `距离国庆节还剩${days}天${hoursDiff}时${minutesDiff}分${secondsDiff}秒`;
        }
    }

    setInterval(countdown, 1000);
</script>

</html>
```

### 反复滚动展示文字
![滚动文字](/src/share/roll.gif "滚动文字")
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>滚动字符效果</title>
    <style>
        #container {
            width: 300px;
            height: 60px;
            border: 1px solid #ccc;
            padding: 10px;
            font-family: monospace;
            overflow: hidden;
        }
        #text {
            margin: 0;
            line-height: 40px;
            white-space: nowrap;
        }
    </style>
</head>
<body>
    <div id="container">
        <p id="text"></p>
    </div>

    <script>
        const fullText = "这是一段文字";
        const minLength = 2;
        let currentLength = minLength;
        let isIncreasing = true;
        const speed = 300;
        const textElement = document.getElementById('text');
        textElement.textContent = fullText.substring(0, minLength);
        function scrollText() {
            if (isIncreasing) {
                currentLength++;
                if (currentLength >= fullText.length) {
                    isIncreasing = false;
                }
            } else {
                currentLength--;
                if (currentLength <= minLength) {
                    isIncreasing = true;
                }
            }
            textElement.textContent = fullText.substring(0, currentLength);
        }
        setInterval(scrollText, speed);
    </script>
</body>
</html>
```

### 注册模拟
![注册表单](/src/share/regForm.png "注册表单")
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8" />
    <title>注册</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        .required::before {
            content: "*";
            color: #df2f30;
            font-weight: bold;
            margin-right: 2px;
        }

        .regTable {
            margin: 0 auto;
            border: 1px solid grey;
            padding: 20px;
            margin-top: 10vh;
            border-radius: 12px;
        }

        .regTable tr {
            height: 80px;
        }

        .regTable input {
            height: 40px;
            width: 320px;
            text-indent: 10px;
            background-color: #f0f0f0;
            vertical-align: middle;
            border: none;
        }

        .regTable tr:nth-child(2) input:nth-child(2) {
            background-color: white;
            border: 1px solid #999999;
            color: grey;
            text-indent: 0;
        }

        .regTable tr:nth-last-child(2) input {
            background-color: #df2f30;
            text-indent: 0;
            color: white;
        }

        .regTable tr:nth-last-child(1) {
            height: 30px;
        }
                .regTable tr:nth-last-child(1) a{
            text-decoration: none;
            color: #004488;
        }

        .regTable #getVerificationCode {
            width: 38%;
        }

        .regTable:first-child {
            text-align: right;
        }

        .regTable #verificationCode {
            width: 59.5%;
        }

        .tip {
            display: inline-block;
            font-size: 12px;
            width: 240px;
            margin: 18px 2.5%;
            text-align: left;
            color: #999999;
            vertical-align: middle;
        }

        .warn {
            color: #df2f30;
        }
    </style>
</head>

<body>
    <form action="" method="" class="regFrom">
        <table class="regTable">
            <tr>
                <td><label class="required">手机号码：</label></td>
                <td><input type="text" name="" id="phone" placeholder="请输入" /></td>
                <td class="tip" id="phoneTip">手机号可用于登录、找回密码等</td>
            </tr>
            <tr>
                <td><label class="required">手机验证码：</label></td>
                <td>
                    <input type="text" name="" id="verificationCode" placeholder="请输入" />
                    <input type="button" name="" id="getVerificationCode" value="获取短信验证码" />
                </td>
                <td class="tip" id="verificationCodeTip"></td>
            </tr>
            <tr>
                <td><label>用户名：</label></td>
                <td><input type="text" name="" id="userName" placeholder="选填" /></td>
                <td class="tip" id="userNameTip">4-20位字符，汉字或英文，或汉字、英文、数字、下划线的任意组合</td>
            </tr>
            <tr>
                <td><label>密码：</label></td>
                <td><input type="password" name="" id="password" placeholder="选填" /></td>
                <td class="tip" id="passwordTip">8-20位，且同时包含数字与大、小写字母</td>
            </tr>
            <tr>
                <td></td>
                <td>
                    <input type="button" name="" id="submitBtn" value="同意以下协议并注册" />
                </td>
            </tr>
            <tr>
                <td></td>
                <td>
                    <a href="">《用户协议》</a>
                    <a href="">《隐私政策》</a>
                    <a href="">《常见问题》</a>
                </td>
            </tr>
        </table>
    </form>
</body>
<script>
    // input对象
    let phoneNum = document.getElementById("phone");
    let verificationCode = document.getElementById("verificationCode");
    let userName = document.getElementById("userName");
    let password = document.getElementById("password");
    // 提示对象
    let phoneNumTip = document.getElementById("phoneTip");
    let verificationCodeTip = document.getElementById("verificationCodeTip");
    let userNameTip = document.getElementById("userNameTip");
    let passwordTip = document.getElementById("passwordTip");
    // 正则表达式
    let phoneNumReg = /^1[3-9]\d{9}$/;
    let verificationCodeReg = /^d{6}$/;
    let userNameReg = /^[\u4e00-\u9fa5a-zA-Z][\u4e00-\u9fa5a-zA-Z0-9_]{3,19}$/;
    let passwordReg = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d]{8,20}$/;

    // 为input对象注册事件
    // 手机号
    phoneNum.onblur = function () {
        if (phoneNumReg.test(this.value)) {
            phoneNumTip.textContent = "手机号合法";
            phoneNumTip.className = "tip";
        } else {
            phoneNumTip.textContent = "手机号不合法";
            phoneNumTip.className = "tip warn";
        }
    };
    phoneNum.onfocus = function () {
        phoneNumTip.className = "tip";
        phoneNumTip.textContent = "手机号可用于登录、找回密码等";
    };
    // 验证码
    verificationCode.onblur = function () {
        if (this.value == "" || this.value == null) {
            verificationCodeTip.textContent = "请输入验证码";
            verificationCodeTip.className = "tip";
            return;
        }
        if (verificationCodeReg.test(this.value)) {
            verificationCodeTip.textContent = "";
            verificationCodeTip.className = "tip";
        } else {
            verificationCodeTip.textContent = "验证码不正确";
            verificationCodeTip.className = "tip warn";
        }
    };
    verificationCode.onfocus = function () {
        verificationCodeTip.className = "tip";
        verificationCodeTip.textContent = "";
    };
    // 用户名
    userName.onblur = function () {
        if (userNameReg.test(this.value)) {
            userNameTip.textContent = "用户名格式正确";
            userNameTip.className = "tip";
        } else {
            userNameTip.textContent =
                "用户名格式错误：4-20位字符，汉字或英文开头，可包含汉字、英文、数字、下划线";
            userNameTip.className = "tip warn";
        }
    };
    userName.onfocus = function () {
        userNameTip.className = "tip";
        userNameTip.textContent = "4-20位字符，汉字或英文，或汉字、英文、数字、下划线的任意组合";
    };
    // 密码
    password.onblur = function () {
        if (passwordReg.test(this.value)) {
            passwordTip.textContent = "密码格式正确";
            passwordTip.className = "tip";
        } else {
            passwordTip.textContent =
                "密码格式错误：8-20位，必须同时包含数字与大、小写字母";
            passwordTip.className = "tip warn";
        }
    };
    password.onfocus = function () {
        passwordTip.className = "tip";
        passwordTip.textContent = "8-20位，且同时包含数字与大、小写字母";
    };

    // 使用alert模拟用户收到验证码，处理验证码被点击的效果
    let getCode = document.getElementById('getVerificationCode');
    let code = null;
    getCode.onclick = function () {
        code = Math.floor(Math.random() * 900000) + 100000;
        alert(code);
    }

    // 仅模拟验证码是否正确的逻辑，后端也有相应的验证码逻辑判断与其他字段的判断并且应该可以向前端相应数据是否被后端接受的回执
    let submitBtn = document.getElementById('submitBtn');
    submitBtn.onclick = function () {
        if (verificationCode.value == code) {
            alert("成功注册");
            // 这里应该会有一个跳转新网页的操作但是我这里直接刷新模拟数据被服务器提交
            location.reload();
        } else {
            alert("验证码错误，请重新获取验证码并重新输入");
            code = null;
            return;
        }
    }
</script>

</html>
```
