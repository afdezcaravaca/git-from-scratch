# 🐢 **<span style="text-decoration: double underline; color:rgba(10,130, 250)">Chapter 0: Git Configuration</span>**

**Author:** Ángel F. Caravaca   
**Date:** 01/06/2026

---

Before starting to use Git and GitHub, you must understand the key difference between the two:

- **Git:** It is a program that tracks code changes. It is installed on your computer.  
- **GitHub:** It is a website used to store Git-managed code online.

From this, you can conclude that there are two different places where your project is stored:

- **Cloud:** The project lives on GitHub, inside a repository. It can be modified directly on the platform, but this is not as comfortable as using a code editor. Depending on the repository's visibility, the code can be seen by everyone or only by you. 

- **Local:** Your project lives on your computer and is modified using your preferred code editor. Only you can see the changes and the evolution of the project. To upload these changes, you must use Git.

Understanding this is crucial because it becomes relevant when explaining branches, merges, and similar concepts. It is essential to know where your code "lives" at any given moment.

Since your code lives in two different places simultaneously, how do you upload local changes to the cloud version of the project? First, you need to configure authentication as that is how you prove to GitHub that you have permission to push changes. Then, you must configure your identity so that all changes are associated with you and everyone knows who to blame for them.

<div align="center">
<img src="https://imgs.search.brave.com/ctDoN-T76JKGoXAXjvJWLgXp9KRU5hKDa7MgQ3lliDM/rs:fit:860:0:0:0/g:ce/aHR0cHM6Ly9tZWRp/YS5nZWVrc2Zvcmdl/ZWtzLm9yZy93cC1j/b250ZW50L3VwbG9h/ZHMvMjAyNTA3MTcx/OTA0MzIxMzQ2NzYv/Z2l0LndlYnA" width=250>
</div>

## 📨 <span style='color:rgba(10,130,250)'><u>HTTPS vs SSH</u></span>

When you execute Git operations such as `clone`, `push`, or `pull`, your local machine and the GitHub servers must connect over the internet. The latter must verify two things before transferring data:

1. **Authentication:** Ensuring that you are who you claim to be.
2. **Authorization:** Ensuring that you have permission to modify the repository.

Git allows you to establish this connection using two different transport protocols, each with its own authentication mechanism. This is the key difference between HTTPS and SSH, and this choice determines *how you connect and authenticate* with the remote server. It is a communication layer, not a content layer, as it does not modify how you use Git at all.

### HTTPS

This is the same protocol that your browser uses, and authentication is based on a credential that you provide. Historically this credential was your password, but GitHub removed it for Git operations in order to make the process safer. Now, a *Personal Access Token* (PAT) is generated in your account: a long string that acts as a revocable credential in place of the password.

To create a PAT you must go to `Settings`, then `Developer Settings`, and then `Personal access tokens`. There you can choose between two types of token, whose key difference lies in how their permissions are defined:

- `Tokens (Classic)`: these are the older type. They let you choose among broad permission categories (*scopes*) but they cannot be restricted to particular repositories: any scope related to your code necessarily applies to all of your repositories at once. This is the sense in which they follow an "all or nothing" criterion. They can also be configured never to expire, which is convenient but carries an obvious risk, since a leaked credential then remains valid indefinitely.

- `Fine-grained tokens`: these are the newer type, created precisely to overcome the limitations of the classic tokens. You have full control over which repositories the token is linked to and which privileges it grants over them, and they are required to have an expiration date. Because of this granularity they are the recommended default for everyone, and an organization can additionally supervise and approve which fine-grained tokens are created within it.

**What is a scope?**   
A scope is a category of permission that answers a single question: what is this token allowed to do with your account? 

Classic tokens rely entirely on this model. Rather than granting unrestricted access, you select from a predefined list of scopes, and the token can do only what those scopes permit. Here, the best practice is to apply the *principle of least privilege*, by which a credential should be granted the minimum access its task requires and nothing more. 

Some scopes act as umbrellas that contain others: `repo` grants full read and write access to both public and private repositories, while `public_repo` is restricted to public ones. This access does not, however, extend to everything: deleting a repository requires the separate `delete_repo` scope, kept apart precisely because the operation is irreversible, and modifying GitHub Actions workflow files requires the `workflow` scope, which `repo` alone does not cover. **The practical rule is to grant only what is strictly necessary**. 

This scope-based approach is what distinguishes classic tokens from fine-grained ones: the latter replace these broad scopes with per-repository *permissions*, letting you confine a token to a single repository and grant it, for example, read-only access to nothing more than its issues.

Once you have defined the token, you use it as follows:

The PAT acts as a direct replacement for your password in any Git operation performed over HTTPS. When you run a command that contacts the remote via an HTTPS URL (for example, when cloning a private repository or running `git push`), Git will prompt you for your credentials in the terminal:

```bash
Username for 'https://github.com': your-github-username
Password for 'https://github.com': # paste your PAT here
```

The crucial point is that, where Git asks for the *Password*, you paste the token rather than your account password, which GitHub no longer accepts for Git operations.

Re-entering the token on every operation would be impractical, so Git provides a *credential helper* that stores it for you. On Linux you have several options, ordered here from least to most secure:

```bash
# In-memory cache: temporary, writes nothing to disk.
# Remembers the token for 15 minutes by default (configurable with --timeout).
git config --global credential.helper cache

# Plain-text store: convenient, but saves the token UNENCRYPTED in ~/.git-credentials. It persists.
git config --global credential.helper store

# System keyring via libsecret (recommended on Linux Mint): stores the token encrypted,
# integrated with the desktop's credential manager. It persists.
git config --global credential.helper libsecret
```

Avoid a practice you will often see online: embedding the token directly in the remote URL (`https://<token>@github.com/user/repo.git`). It works, but it leaves the token in plain text inside the repository configuration and in your shell history, which defeats much of its purpose.

To summarise the procedure:

<div align="center">
<img src="./images/FlowchartHTTPS.png" width=1000>
</div>

### SSH

The Secure Shell (SSH) protocol authenticates you using asymmetric, or public-key, cryptography. In contrast to the HTTPS protocol, where you must type in a credential, here you generate a *key pair*, the **private** and **public keys**, that are asymmetrically related. This means that what is signed with the private key can be verified with the public key, but not the other way around. In this protocol, the part you must keep safe is the private key: the public key is derived from it, but the reverse is computationally infeasible, so the private key cannot be deduced from the public one.

- **Public key (`id_ed25519.pub`):** its role is to **verify** you. You register it on GitHub, which then uses it to check the response produced by your private key and confirm that it came from the matching key.
  
- **Private key (`id_ed25519`):** its role is to **prove** your identity. It never leaves your machine and is what answers GitHub's random challenge by signing it. Only the holder of this key can produce a valid response.

Notice that, during this procedure, your private key never travels over the internet: GitHub sends the challenge, your private key solves it locally and sends back only the answer, and then the public key stored on GitHub verifies it. Moreover, once configured, SSH requires no interactive credential entry on each operation.

To configure the SSH protocol on GitHub, first check whether you already have a key and, if not, generate one:

```bash
ls -al ~/.ssh                                       # lists existing keys (look for id_ed25519 and id_ed25519.pub)
ssh-keygen -t ed25519 -C "name/email/whatever"   # generates a new key pair
```

When prompted, accept the default location (`~/.ssh`) and optionally set a passphrase. Then display your public key and copy its full contents:

```bash
cat ~/.ssh/id_ed25519.pub
```

Go to `Settings` $\rightarrow$ `SSH and GPG keys` $\rightarrow$ `New SSH key`, paste the value, and save it. Finally, verify that everything works:

```bash
ssh -T git@github.com
```

If the configuration is correct, GitHub replies with:
```bash
Hi <usernamer>! You've successfully authenticated, but GitHub does not provide shell access.
```

To summarise the procedure:

<div align='center'>
<img src="./images/FlowchartSSH.png" width=1000>
</div>

> [!NOTE]
> Both protocols are perfectly valid, so the choice is just a matter of preference. I lean towards SSH, since once it is set up it is completely frictionless — there is no token to paste and none to renew — and I think that, as long as you protect your private key and keep your account secure, SSH is perfectly safe for personal use. For day-to-day work I find that combination of convenience and security hard to beat, so SSH is my default.

> [!WARNING]
> If you configure the SSH channel, you must clone the repo using its SSH URL.
> If you configure the HTTPS channel, you must clone the repo using its HTTPS URL.
> You can change how you communicate with the repo:
>
> ```bash
> git remote set-url origin git@github.com:user/repo.git # to SSH
> git remote set-url origin https://github.com/user/repo.git  # to HTTPS
> ```

### User Name and Email

To finish configuring GitHub on your computer, you must set your `user.name` and `user.email`. These are the two values Git stamps onto every commit, so that each change is permanently associated with an author. 

This is the *identity* side of GitHub required along with the *authentication*: not whether you are allowed to push, but who actually made the change. You set them once with:

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

The `--global` flag applies this identity to every repository on your machine. If you need a different identity for a specific project, run the same commands without the `--global` flag from within that repository. This configures a **local** `user.name` and `user.email` that overrides the global settings for that specific repo.

As a best practice, ensure the email matches one registered to your GitHub account, as this links your commits to your profile. If it does not match, the commits will still be created, but GitHub will not attribute them to you.

### GPG keys (optional)

So far, in the previous two sections, we have discussed the HTTPS and SSH protocols. Both answer the same question: **are you allowed to push to this repository?** Now we turn to the GPG key, which answers a different one: **was it really you who made this change?**

The GPG key grants no access at all. Instead, it lets you cryptographically *sign* your commits, proving that they were genuinely written by you. Note that, for day-to-day personal use, this is unnecessary, but it becomes valuable in professional or collaborative settings. For instance, you'll need it for some open-source projects or teams whose policies require that the authorship of every commit be verifiable.

The reason this matters goes back to how authorship is recorded. Every commit is stamped with the name and email you set through `user.name` and `user.email`, but those are merely text fields that anyone can fill with any value. Nothing prevents a third party from creating a commit that appears, on its face, to have been authored by you. Signing a commit with your private GPG key closes that gap: GitHub verifies the signature against the public GPG key you have registered and, when it matches, marks the commit as **Verified**. Put briefly, `user.name` and `user.email` *declare* who you are, whereas a GPG signature *proves* it.

Like SSH, GPG relies on a key pair: a private key that stays on your machine and does the signing, and a public key that you upload to GitHub for verification. The setup is as follows.

To configure your GPG key, first, check whether you already have a key, and otherwise generate one:

```bash
gpg --list-secret-keys --keyid-format=long   # lists existing keys
gpg --full-generate-key                       # generates a new one (interactive)
```

During generation, accept the default algorithm, set an expiration date if you wish, and use the same email address you use on GitHub, so the signature can be tied to your account.

Next, locate the long identifier of your key and export its public part:

```bash
# The key ID is the value after the algorithm.
gpg --list-secret-keys --keyid-format=long

# Export the public key in the text format GitHub expects:
gpg --armor --export YOUR_KEY_ID
```

Copy the entire block, including the `-----BEGIN PGP PUBLIC KEY BLOCK-----` and `-----END...-----` lines, and add it on GitHub under `Settings` $\rightarrow$ `SSH and GPG keys` $\rightarrow$ `New GPG key`.

Finally, tell Git which key to use and to sign your commits automatically:

```bash
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true   # sign every commit by default

# On Linux, this allows GPG to prompt for your passphrase in the terminal:
export GPG_TTY=$(tty)
```
From then on your commits will be signed, and you can confirm it by running `git log --show-signature` locally or by looking for the **Verified** label beside the commit on GitHub. 

If you would rather not sign every commit, omit `commit.gpgsign` and instead add the `-S` flag to the ones you choose: `git commit -S -m "message"`.
