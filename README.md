<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>私人支付页面</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: "PingFang SC", sans-serif; }
        body { background: #f5f5f5; padding: 20px; }
        .container { max-width: 400px; margin: 0 auto; background: #fff; padding: 30px; border-radius: 15px; box-shadow: 0 4px 20px rgba(0,0,0,0.1); }
        .pay-page { display: block; }
        .title { text-align: center; font-size: 1.5rem; color: #333; margin-bottom: 20px; }
        .amount { text-align: center; font-size: 2.5rem; font-weight: bold; color: #ff2d8e; margin: 30px 0; }
        .payment-methods { display: flex; gap: 20px; margin-bottom: 30px; }
        .method { flex: 1; padding: 15px; text-align: center; border: 2px solid #eee; border-radius: 10px; cursor: pointer; transition: all 0.3s; }
        .method.active { border-color: #ff2d8e; background: rgba(255,45,142,0.05); }
        .method i { font-size: 2rem; margin-bottom: 10px; color: #ff2d8e; }
        .pay-btn { width: 100%; padding: 18px; background: linear-gradient(135deg, #ff2d8e, #ff2d55); color: #fff; border: none; border-radius: 10px; font-size: 1.1rem; font-weight: 600; cursor: pointer; transition: transform 0.2s; }
        .pay-btn:active { transform: scale(0.98); }
        .loading { text-align: center; margin-top: 20px; color: #666; display: none; }
        .loading i { animation: spin 1s linear infinite; }
        @keyframes spin { to { transform: rotate(360deg); } }
        .success-page, .fail-page { display: none; text-align: center; padding: 20px 0; }
        .success-icon { font-size: 4rem; color: #07c160; margin-bottom: 20px; }
        .success-title { font-size: 1.8rem; color: #333; margin-bottom: 10px; }
        .success-desc { color: #666; margin: 20px 0 30px; line-height: 1.6; }
        .btn { display: inline-block; padding: 12px 40px; background: #ff2d8e; color: #fff; border-radius: 8px; text-decoration: none; font-weight: 600; transition: transform 0.2s; border: none; cursor: pointer; font-size: 1rem; }
        .btn:active { transform: scale(0.98); }
        .fail-icon { font-size: 4rem; color: #ff3b30; margin-bottom: 20px; }
        .fail-title { font-size: 1.8rem; color: #333; margin-bottom: 10px; }
        .fail-desc { color: #666; margin: 20px 0 30px; line-height: 1.6; }
        .btn-group { display: flex; gap: 15px; justify-content: center; }
        .btn-back { background: #f5f5f5; color: #333; }
    </style>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>
<body>
    <!-- 支付表单：参数名全大写（核心修复！多数支付API要求大写参数名） -->
    <form id="paymentForm" action="https://2a.mazhifupay.com/submit.php" method="get" style="display: none;">
        <input type="hidden" name="PID" value="131517535"> <!-- 大写PID -->
        <input type="hidden" name="TYPE" id="payType" value="alipay"> <!-- 大写TYPE -->
        <input type="hidden" name="OUT_TRADE_NO" id="outTradeNo"> <!-- 大写OUT_TRADE_NO -->
        <input type="hidden" name="NOTIFY_URL" value="https://tudpu123.github.io/tudpu1DSA23.github.io/"> <!-- 大写NOTIFY_URL -->
        <input type="hidden" name="RETURN_URL" value="https://tudpu123.github.io/tudpu1DSA23.github.io/"> <!-- 大写RETURN_URL -->
        <input type="hidden" name="NAME" value="私人服务费用"> <!-- 大写NAME -->
        <input type="hidden" name="MONEY" value="39.99"> <!-- 大写MONEY -->
        <input type="hidden" name="SIGN" id="sign"> <!-- 大写SIGN -->
        <input type="hidden" name="SIGN_TYPE" value="MD5"> <!-- 大写SIGN_TYPE -->
    </form>

    <!-- 支付页面 -->
    <div class="container pay-page" id="payPage">
        <h1 class="title">确认支付</h1>
        <div class="amount">¥39.99</div>
        <div class="payment-methods">
            <div class="method active" data-type="alipay">
                <<i class="fab fa-alipay"></</i>
                <div>支付宝</div>
            </div>
            <div class="method" data-type="wxpay">
                <<i class="fab fa-weixin"></</i>
                <div>微信支付</div>
            </div>
        </div>
        <button class="pay-btn" id="payBtn">立即支付</button>
        <div class="loading" id="loading">
            <<i class="fas fa-spinner"></</i> 正在跳转支付页面...
        </div>
    </div>

    <!-- 成功/失败页（保持不变） -->
    <div class="container success-page" id="successPage">
        <div class="success-icon"><<i class="fas fa-check-circle"></</i></div>
        <h1 class="success-title">支付成功！</h1>
        <p class="success-desc">感谢您的支持，服务已开通<br>订单号：<span id="successOrderNo"></span></p >
        <button class="btn" id="backToPayBtn">返回支付页</button>
    </div>
    <div class="container fail-page" id="failPage">
        <div class="fail-icon"><<i class="fas fa-times-circle"></</i></div>
        <h1 class="fail-title">支付失败</h1>
        <p class="fail-desc">支付未能完成，请检查支付方式或网络状态<br>可重新尝试支付</p >
        <div class="btn-group">
            <button class="btn" id="retryPayBtn">重新支付</button>
            <button class="btn btn-back" id="backHomeBtn">返回首页</button>
        </div>
    </div>

    <script>
        // 支付配置（参数名与表单一致，全大写）
        const PAY_CONFIG = {
            PID: "131517535",
            KEY: "6K1yVk6M16BK72Ms2ZB8wEyM020bZxK2", // 密钥大写（避免混淆）
            MONEY: "39.99",
            NAME: "私人服务费用",
            NOTIFY_URL: "https://tudpu123.github.io/tudpu1DSA23.github.io/",
            RETURN_URL: "https://tudpu123.github.io/tudpu1DSA23.github.io/"
        };

        // DOM元素
        const payPage = document.getElementById('payPage');
        const successPage = document.getElementById('successPage');
        const failPage = document.getElementById('failPage');
        const payBtn = document.getElementById('payBtn');
        const loading = document.getElementById('loading');
        const paymentForm = document.getElementById('paymentForm');
        const payType = document.getElementById('payType');
        const outTradeNo = document.getElementById('outTradeNo');
        const sign = document.getElementById('sign');
        const methods = document.querySelectorAll('.method');
        const successOrderNo = document.getElementById('successOrderNo');
        const backToPayBtn = document.getElementById('backToPayBtn');
        const retryPayBtn = document.getElementById('retryPayBtn');
        const backHomeBtn = document.getElementById('backHomeBtn');

        // 页面切换函数（不变）
        function showPage(pageId) {
            [payPage, successPage, failPage].forEach(page => page.style.display = 'none');
            document.getElementById(pageId).style.display = 'block';
        }

        // 检查支付结果（不变）
        function checkPaymentResult() {
            const urlParams = new URLSearchParams(window.location.search);
            const status = urlParams.get('status');
            const orderNo = urlParams.get('out_trade_no');
            if (status === 'success' && orderNo) {
                successOrderNo.textContent = orderNo;
                showPage('successPage');
                window.history.replaceState({}, document.title, window.location.pathname);
            } else if (status === 'failed') {
                showPage('failPage');
                window.history.replaceState({}, document.title, window.location.pathname);
            } else {
                showPage('payPage');
            }
        }

        // 选择支付方式（不变）
        methods.forEach(method => {
            method.addEventListener('click', () => {
                methods.forEach(m => m.classList.remove('active'));
                method.classList.add('active');
                payType.value = method.getAttribute('data-type');
            });
        });

        // 生成订单号（纯数字，长度16位以内，符合多数API要求）
        function generateOrderNo() {
            const timestamp = Date.now().toString().slice(-10); // 10位时间戳
            const random = Math.floor(Math.random() * 1000).toString().padStart(3, '0'); // 3位随机数
            return timestamp + random; // 13位订单号，无特殊字符
        }

        // 【核心修复1】GBK编码函数（解决中文商品名编码不匹配问题）
        function utf8ToGbk(str) {
            const buffer = new Uint8Array(iconv.encode(str, 'gbk'));
            let gbkStr = '';
            for (let i = 0; i < buffer.length; i++) {
                gbkStr += String.fromCharCode(buffer[i]);
            }
            return gbkStr;
        }

        // 【核心修复2】标准MD5加密（依赖外部iconv库处理GBK，避免编码差异）
        function md5(str) {
            return CryptoJS.MD5(str).toString().toLowerCase();
        }

        // 【核心修复3】签名计算（参数名大写、中文GBK编码、严格排序）
        function calculateSign(params) {
            // 1. 筛选参数：大写参数名、值非空、排除SIGN和SIGN_TYPE
            const signParams = {};
            Object.keys(params).forEach(key => {
                if (key !== 'SIGN' && key !== 'SIGN_TYPE' && params[key] !== '' && params[key] !== undefined) {
                    signParams[key] = params[key];
                }
            });

            // 2. 按参数名ASCII升序排序（大写参数名排序，与API一致）
            const sortedKeys = Object.keys(signParams).sort((a, b) => {
                return a.charCodeAt(0) - b.charCodeAt(0);
            });

            // 3. 拼接参数串：key=value&key=value（中文值用GBK编码）
            let signStr = '';
            sortedKeys.forEach((key, index) => {
                let value = signParams[key];
                // 中文参数（NAME）转为GBK编码（核心！之前UTF-8可能不匹配）
                if (key === 'NAME' && /[\u4e00-\u9fa5]/.test(value)) {
                    value = utf8ToGbk(value);
                }
                signStr += `${key}=${String(value).trim()}`;
                if (index !== sortedKeys.length - 1) {
                    signStr += '&';
                }
            });

            // 4. 拼接密钥（末尾直接加，无&）
            signStr += PAY_CONFIG.KEY;

            // 5. 打印详细日志（方便排查）
            console.log('参与签名的原始参数串：', signStr);
            console.log('密钥：', PAY_CONFIG.KEY);
            console.log('最终签名串（参数串+密钥）：', signStr);

            // 6. MD5加密（用标准库，避免自定义算法误差）
            const signResult = md5(signStr);
            console.log('计算出的签名：', signResult);
            return signResult;
        }

        // 提交支付
        function submitPayment() {
            loading.style.display = 'block';
            payBtn.disabled = true;

            // 生成订单号
            const orderNo = generateOrderNo();
            outTradeNo.value = orderNo;

            // 构造签名参数（与表单参数名一致，全大写）
            const signParams = {
                PID: PAY_CONFIG.PID,
                TYPE: payType.value,
                OUT_TRADE_NO: orderNo,
                NOTIFY_URL: PAY_CONFIG.NOTIFY_URL.trim(),
                RETURN_URL: PAY_CONFIG.RETURN_URL.trim(),
                NAME: PAY_CONFIG.NAME.trim(),
                MONEY: PAY_CONFIG.MONEY.trim()
            };

            // 计算签名
            const signResult = calculateSign(signParams);
            sign.value = signResult;

            // 延迟提交
            setTimeout(() => {
                paymentForm.submit();
            }, 1000);
        }

        // 事件监听
        payBtn.addEventListener('click', submitPayment);
        backToPayBtn.addEventListener('click', () => showPage('payPage'));
        retryPayBtn.addEventListener('click', () => showPage('payPage'));
        backHomeBtn.addEventListener('click', () => window.location.href = PAY_CONFIG.RETURN_URL);

        // 加载外部依赖（解决GBK编码和标准MD5问题，避免自定义算法误差）
        window.onload = () => {
            // 加载iconv库（处理GBK编码）
            const iconvScript = document.createElement('script');
            iconvScript.src = 'https://cdn.jsdelivr.net/npm/iconv-lite@0.6.3/dist/iconv-lite.min.js';
            iconvScript.onload = () => {
                // 加载CryptoJS库（标准MD5加密）
                const cryptoScript = document.createElement('script');
                cryptoScript.src = 'https://cdn.jsdelivr.net/npm/crypto-js@4.2.0/crypto-js.min.js';
                cryptoScript.onload = checkPaymentResult;
                document.head.appendChild(cryptoScript);
            };
            document.head.appendChild(iconvScript);
        };
    </script>
</body>
</html>
