# Managing Jdks and Certificates efficiently

## Context
Usually, we need to add a certificate to the truststore in a company environment to be able to use the jdk access to the internet. Things start to get complicated when we start to have several jdks version, as the truststore is generally link to one jdk. 

Some people use tools like `sdkman!`, but I prefer to have a cleaner "system-level" feel.
the best approach is to use a Global Version Manager combined with a Shared Truststore.

---
## TL;DR
### 1. Version Management: **jEnv**
It simply manages the JDKs you already have.

* **Install**: `brew install jenv`

* **Add**: `jenv add /path/to/jdk_folder`

* **Use**: `jenv global 21` or `jenv local 17` (per project).

**Vital**: Enable the export plugin to keep your `$JAVA_HOME` in sync: `jenv enable-plugin export`

### 2. The Certificate Fix: Global Truststore
Stop modifying every new JDK. Force Java to use one single certificate file for the whole system.

**Setup**: Copy an existing `cacerts` file to `~/.java/security/global-cacerts`.

**Import**: Add your 3rd party certs once to this file.

**Activate**: Add this to your .zshrc or .bashrc:
```
export JAVA_TOOL_OPTIONS="-Djavax.net.ssl.trustStore=$HOME/.java/security/global-cacerts -Djavax.net.ssl.trustStorePassword=changeit"
```

### 3. IntelliJ Configuration
JDKs: Go to File > Project Structure and add the JDKs located in ~/.jenv/versions/.

**Automation**: IntelliJ will inherit your JAVA_TOOL_OPTIONS from your shell, so your certificates will work instantly in the IDE without per-project configuration.

**In short**: jEnv handles the paths, and JAVA_TOOL_OPTIONS handles the security centrally.

---

## 1. Managing different JDKs
### The Best Tool: jenv and asdf
If SDKMAN felt too "intrusive" or messy, asdf is the industry standard for managing multiple runtimes (Java, Node, Python, etc.) via a single tool.

#### jenv (for macOS/Linux): 
If you like to download JDKs manually (from Oracle, Azul, etc.) and just want a way to switch JAVA_HOME easily, use jEnv. You just "add" your existing JDK paths to it, and it manages the switching.


#### using asdf
It uses a simple `.tool-versions` file in your project folders to switch versions automatically when you cd into them.

#### Installation:
- Install asdf via their official guide.
- Add the Java plugin: asdf plugin add java.
- Install a version: asdf install java openjdk-21.

Set it globally: `asdf global java openjdk-21`.

---

## 2. Managing certificates

### Solving the "Certificate Annoyance"
The reason you have to re-configure certificates is that every JDK comes with its own cacerts file located inside its installation folder. When you switch JDKs, you switch to a "blank" truststore.

The "One Truststore to Rule Them All" Strategy
Instead of importing certificates into every JDK, create one Global Truststore and tell all your JDKs to use it.

Create a central truststore: Copy the cacerts file from one of your JDKs to a permanent location, e.g., `~/.java/global-cacerts`.

#### Import your certificates once:

```
keytool -import -trustcacerts -keystore ~/.java/global-cacerts -storepass changeit -alias my-api -file my_cert.crt
```

You can also just copy the cacert that you create on a specific jdk to this place (don't need to renegerate it).

```
cp /path/to/your/current/working/truststore ~/.java/global-cacerts
```

####  If your password is NOT 'changeit', update this:
export JAVA_TOOL_OPTIONS="-Djavax.net.ssl.trustStore=$HOME/.java/security/global-cacerts -Djavax.net.ssl.trustStorePassword=YOUR_PASSWORD"

#### Merging vs. Replacing
If your existing file is a full cacerts copy: (meaning it has your private certs plus the standard internet ones like DigiCert/Verisign), then you are good to go.

If your existing file ONLY contains your private certs: You might run into issues connecting to standard websites (like Maven Central or GitHub) because the "official" root certificates are missing.

**How to verify what's inside**: 

Run this command to see how many certificates are in your file: 
```
keytool -list -keystore ~/.java/security/global-cacerts -storepass yourpassword | grep "Your keystore contains"
```

* ~100+ certs: It’s a full bundle (Safe to use as-is).

* 1-5 certs: It’s a private-only store. In this case, you should import your certs into a fresh copy of the system cacerts instead.

---

Point all Java processes to it: Add this environment variable to your `.zshrc` or `.bashrc`:

```
export JAVA_TOOL_OPTIONS="-Djavax.net.ssl.trustStore=$HOME/.java/global-cacerts -Djavax.net.ssl.trustStorePassword=changeit"
```

**Result** : No matter which JDK you switch to (via asdf, jEnv, or even IntelliJ), the JVM will automatically pick up your custom certificates from that one file.

Why this is better for jEnv
Since jenv works by changing your PATH and JAVA_HOME, it doesn't actually change your shell environment variables like JAVA_TOOL_OPTIONS.

**By pointing to this single file, you bypass the "Java 11 has its own certs, Java 21 has its own certs" problem entirely.**

Every version managed by jenv will wake up, read that specific file, and "just work."

---

### 3. Connecting IntelliJ to this Setup
Since you don't like IntelliJ's internal management, you can point it to use your system-managed JDKs:

Go to File > Project Structure > SDKs.

Click + and select Add JDK....

Point it to the directory where asdf or your manual install keeps the JDKs (usually ~/.asdf/installs/java/...).

Because you set the JAVA_TOOL_OPTIONS globally in your shell profile, IntelliJ will usually inherit that setting if you launch it from the terminal or have configured your OS environment variables correctly.