---
title: Java 集成 Google Authenticator 实现两步验证实战
date: 2025-10-29 17:30:25
category: 后端
tags: 综合技术
---

两步验证（2FA）是提升账户安全性的有效手段，在输入密码后，还需提供一种仅用户本人持有的动态凭证。Google Authenticator (GA)
作为一款广泛使用的基于时间的一次性密码（TOTP）生成器，其算法是开放的，可以方便地集成到我们自己的应用中。

基本原理：TOTP 算法
Google Authenticator 的核心是基于时间的动态令牌算法，它是 HOTP（基于HMAC的一次性密码） 算法的一种扩展。

共享密钥：服务端生成一个Base32编码的随机密钥，并安全地分享给用户（通常通过二维码）。用户将此密钥存入手机 Authenticator APP。

时间切片：算法将当前时间（Unix 时间戳，1970年以来的秒数）除以一个时间窗口（默认 30秒），得到一个不断增长的计数 C。

HMAC 计算：使用共享密钥和计数 C 作为输入，通过 HMAC-SHA1 算法生成一个哈希值。

动态截取：从哈希值中动态截取一段，生成一个 6位（或8位）数字。这就是我们看到的、每30秒变化一次的验证码。

核心等式：TOTP = Truncate(HMAC-SHA1(SecretKey, CurrentTime / TimeStep))

开发流程概览
服务端为用户生成密钥。

生成二维码：将密钥、用户标识、发行者等信息按照特定格式（otpauth:// URL）编码成二维码，供用户 APP 扫描绑定。

用户绑定：用户使用 Google Authenticator 或类似 APP（如 Microsoft Authenticator, Authy）扫描二维码，APP 将密钥保存。

验证：用户登录时，输入 APP 上显示的6位数字。服务端用相同的密钥和当前时间，运行相同的 TOTP
算法，生成一个验证码。如果用户输入的码与服务端计算的码在允许的时间漂移范围内匹配，则验证通过。

Java 实现实战
我们无需自己实现底层算法，可以使用现成的库，例如 com.warrenstrange:googleauth。

1. 添加 Maven 依赖

xml
<dependency>
<groupId>com.warrenstrange</groupId>
<artifactId>googleauth</artifactId>
<version>1.5.0</version> <!-- 请检查最新版本 -->
</dependency>

2. 服务端代码示例

java
import com.warrenstrange.googleauth.GoogleAuthenticator;
import com.warrenstrange.googleauth.GoogleAuthenticatorKey;
import com.warrenstrange.googleauth.GoogleAuthenticatorQRGenerator;

public class GoogleAuthService {

    private final GoogleAuthenticator gAuth = new GoogleAuthenticator();

    /**
     * 为用户生成新的密钥和绑定信息
     * @param username 用户名
     * @param issuer 发行者名称（如你的公司或应用名）
     * @return 包含密钥和二维码URL的对象
     */
    public AuthKeyInfo generateKey(String username, String issuer) {
        // 1. 生成密钥
        final GoogleAuthenticatorKey key = gAuth.createCredentials();
        String secretKey = key.getKey(); // 这是Base32编码的密钥

        // 2. 生成供扫描的二维码URL (otpauth://totp/...)
        String qrCodeUrl = GoogleAuthenticatorQRGenerator.getOtpAuthTotpURL(
                issuer, username, new GoogleAuthenticatorKey.Builder(secretKey).build()
        );
        // 在实际应用中，你需要将此URL生成二维码图片（可使用例如ZXing库）
        // String qrCodeImageData = "data:image/png;base64," + generateBase64QRCode(qrCodeUrl);

        return new AuthKeyInfo(secretKey, qrCodeUrl);
    }

    /**
     * 验证用户输入的验证码
     * @param secretKey 该用户绑定的Base32密钥
     * @param verificationCode 用户输入的6位数字
     * @return 验证是否成功
     */
    public boolean verifyCode(String secretKey, int verificationCode) {
        // 此方法内部会处理时间窗口漂移（通常允许前后一个时间窗口，即±30秒）
        return gAuth.authorize(secretKey, verificationCode);
    }

    // 存储密钥信息的简单DTO
    public static class AuthKeyInfo {
        private final String secretKey;
        private final String qrCodeUrl;
        // getters...
    }

}

3. 在业务逻辑中集成

java
@RestController
@RequestMapping("/api/2fa")
public class TwoFactorAuthController {

    @Autowired
    private GoogleAuthService googleAuthService;
    @Autowired
    private UserService userService; // 假设的用户服务

    /**
     * 为用户开启2FA，返回密钥和二维码
     */
    @PostMapping("/enable")
    public ResponseEntity<?> enable2FA(@CurrentUser User user) {
        // 检查是否已开启
        // 生成密钥信息
        GoogleAuthService.AuthKeyInfo keyInfo = googleAuthService.generateKey(user.getUsername(), "MyAwesomeApp");
        // !!重要：必须将 secretKey 安全地与该用户关联存储到数据库！！
        userService.save2FASecret(user.getId(), keyInfo.getSecretKey());
        // 返回二维码URL给前端展示，前端需将其渲染为二维码图片
        return ResponseEntity.ok(Map.of(
            "secretKey", keyInfo.getSecretKey(), // 通常仅用于手动输入备份，可不返回给前端
            "qrCodeUrl", keyInfo.getQrCodeUrl()
        ));
    }

    /**
     * 验证并激活2FA（用户首次扫描后输入验证码确认）
     */
    @PostMapping("/verify-activation")
    public ResponseEntity<?> verifyAndActivate(@CurrentUser User user,
                                               @RequestParam int code) {
        String storedSecret = userService.get2FASecret(user.getId());
        if (storedSecret == null) {
            return ResponseEntity.badRequest().body("请先获取2FA设置信息");
        }
        boolean isValid = googleAuthService.verifyCode(storedSecret, code);
        if (isValid) {
            userService.activate2FA(user.getId()); // 标记用户已激活2FA
            return ResponseEntity.ok("两步验证已成功激活");
        } else {
            return ResponseEntity.badRequest().body("验证码无效");
        }
    }

    /**
     * 登录时的2FA验证步骤
     */
    @PostMapping("/authenticate")
    public ResponseEntity<?> authenticate(@RequestParam String username,
                                          @RequestParam int code) {
        // 1. 先验证用户名密码...
        // 2. 密码验证通过后，进行2FA验证
        User user = userService.findByUsername(username);
        String storedSecret = userService.get2FASecret(user.getId());
        if (storedSecret != null && googleAuthService.verifyCode(storedSecret, code)) {
            // 2FA验证成功，生成登录令牌
            String token = jwtUtil.generateToken(user);
            return ResponseEntity.ok(Map.of("token", token));
        }
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("两步验证失败");
    }

}
关键注意事项
密钥存储安全：生成的 secretKey 必须像密码一样安全地存储（建议加密后存入数据库），并与用户ID强关联。

备份与恢复：必须提供备用验证码（Recovery Codes），在用户丢失手机时用于紧急登录。这些码应在用户开启2FA时生成并安全地交给用户保存。

时间同步：服务器和用户手机的时间必须大致同步，否则会导致验证失败。库通常允许一定的时间窗口漂移。

用户体验：清晰的引导用户完成扫描绑定和首次验证的流程。对于不支持扫码的环境，应提供手动输入密钥的选项。

通过以上步骤，你可以在自己的 Java 应用中成功集成基于 TOTP 的两步验证功能，显著提升账户安全性。