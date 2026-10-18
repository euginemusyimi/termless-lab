# Before every push

- [ ] No passwords, setup keys, API tokens, private keys, `.ovpn` files or OPNsense `config.xml` (these contain secrets)
- [ ] Screenshots cropped: no public IP, tokens, tenant IDs, MAC addresses you care about
- [ ] Private 10.x addresses are fine; your real public IP and home address are not
- [ ] Anything that was ever pasted publicly has been rotated (assume it is compromised)
- [ ] `mkdocs build --strict` passes locally
- [ ] Each debrief says what is *verified* vs. *unknown* (honest root causes read better than tidy ones)
- [ ] Employer, client or student data: none
