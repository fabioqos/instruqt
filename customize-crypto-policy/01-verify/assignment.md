---
slug: verify
id: awwfmbqjysxv
type: challenge
title: Verify the current system-wide cryptographic policy setting
notes:
- type: text
  contents: |
    # Goal:
    After completing this scenario, you will be able to customize Red Hat Enterprise Linux's system-wide cryptographic policies.  The scenario uses the Apache web server
    as the example application.

    # Concepts included in this scenario:
    * Verify the current system-wide cryptographic policy setting
    * Make customizations to cryptography methods included in a policy
    * Restart applications after applying a policy modifier to named policy
    * Verify that the application settings comply with the modified cryptographic policies

    # Example Use case:
    Your security team wants to start using FUTURE crypto policy, but the applications still need time to catch up. Your team needs to modify the FUTURE crypto policy to balance the needs of the security and application teams.
tabs:
- title: Terminal
  type: terminal
  hostname: rhel
  cmd: tmux attach-session -t "rhel-session" > /dev/null 2>&1
difficulty: basic
timelimit: 3000
---
## Modify the FUTURE crypto policy

The Chief Security Officer sends out the following e-mail -
<pre class="file">
Application and Infrastructure Administrators,

After my last e-mail recommending 3072 bit public keys, I have received few concerns that some applications would need additional time for migration.

To continue supporting these applications running on our platform, and to provide more time for these applications to upgrade, my recommendation is to disallow TLS (1.0, and 1.1), and not allow SHA-1 hash usage.

**NOTE** We should still allow 2048 bit ciphers usage for a certain period of time until all applications can be upgraded to use 3072 bit keys.

-CSO
</pre>

In order to comply with the requirements set forth by the CSO above, you will update the system to modify the **FUTURE** policy to support shorter keys of 2048 bit length. By default, the minimum key length in the **FUTURE** policy is set to 3072-bit.

You will now check the currently active crypto policy in effect on the system -

```bash
update-crypto-policies --show
```

<pre class="file">
root@rhel:~# update-crypto-policies --show
FUTURE
</pre>

Restart Apache.

```bash
systemctl restart httpd.service
```

<pre class="file">
root@rhel:~# systemctl restart httpd.service
Job for httpd.service failed because the control process exited with error code.
See "systemctl status httpd.service" and "journalctl -xeu httpd.service" for details.
</pre>

The Apache service has failed to start. You will now check the status of the Apache service -

```bash
systemctl status httpd.service --no-pager
```

<a href="#example_image">
 <img alt="An example image" src="../assets/httpderror.png" />
</a>

<a href="#" class="lightbox" id="example_image">
 <img alt="An example image" src="../assets/httpderror.png" />
</a>

You can see a more specific error message in the SSL error log for Apache.

```bash
tail -2 /var/log/httpd/ssl_error_log
```

<pre class="file">
[Tue Jul 16 15:13:25.580860 2019 ] [ssl:emerg] [pid 8869:tid 140233336588544] AH02562: Failed to configure certificate fe80::42:acff:fe11:b:443:0 (with chain), check /etc/pki/tls/certs/localhost.crt
[Tue Jul 16 15:13:25.580860 2019 ] [ssl:emerg] [pid 8869:tid 140233336588544] SSL Library Error: error: 140AB18F: SSL routines: SSL_CTX_use_certificate:ee key too small
</pre>

The error message indicates that the key length was too small which was expected because
the **FUTURE** policy requires a minimum of 3072-bit keys.

Now, you will create a policy modifier module called **2048KEYS.pmod** that can be used with the FUTURE crypto policy. Policy modifiers are text files that include policy instructions to the update-crypto-policies tool.

The naming of these files must follow the following convention : **_MODULE_.pmod**, where **_MODULE_** is the name of the modifier in uppercase without spaces, and .pmod is the file extension in lowercase.

Next, you will create a policy modifier called **2048KEYS.pmod** that will set the minimum key size to 2048 bits.

```bash
touch /etc/crypto-policies/policies/modules/2048KEYS.pmod
```

In the policy modifier file, you will specify the minimum key size of RSA and DH keys -

```bash
echo "min_dh_size = 2048" > /etc/crypto-policies/policies/modules/2048KEYS.pmod
```

```bash
echo "min_rsa_size = 2048" >> /etc/crypto-policies/policies/modules/2048KEYS.pmod
```

You will now configure the system to use a modified **FUTURE** policy with our newly created policy modifier -

```bash
update-crypto-policies --set FUTURE:2048KEYS
```

If you want to apply multiple policy modifiers, you can chain together several policy modifiers separated using ':'.
The policy modifiers are evaluated left to right to modify the specified named policy.

<pre class="file">
Setting system policy to FUTURE:2048KEYS
Note: System-wide crypto policies are applied on application start-up.
It is recommended to restart the system for the change of policies
to fully take place.
</pre>

You can now verify that the new policy has been applied to the system.

```bash
update-crypto-policies --show
```

<pre class="file">
FUTURE:2048KEYS
</pre>

<style>
.lightbox {
  display: none;
  position: fixed;
  justify-content: center;
  align-items: center;
  z-index: 999;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  padding: 1rem;
  background: rgba(0, 0, 0, 0.8);
}

.lightbox:target {
  display: flex;
}

.lightbox img {
  max-height: 100%;
}
</style>
