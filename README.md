# andrews-stuff
Helpful scripts and links that Andrew likes to use.

## GitHub Copilot

- [Use Copilot sandbox with encrypted token on Windows](./copilot-sandbox-windows.md)

## Parallels
Disable keyboard sync between Mac and Windows: [How to disable keyboard layout synchronisation between Mac and Windows virtual machine.](https://kb.parallels.com/115200)

## C-Sharp
[CleanupCode Command-Line Tool | JetBrains Rider Documentation](https://www.jetbrains.com/help/rider/CleanupCode.html)

### Web Security and OWASP
- [OWASP WebGoat | OWASP Foundation](https://owasp.org/www-project-webgoat/)

CSP:
- [Content Security Policy (CSP) - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)
- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)

### SQL Server LocalDB on Windows for ARM
```bash
SqlLocalDB.exe start MSSQLLocalDB
SqlLocalDB.exe info MSSQLLocalDB

# Connection String format
"Data Source=np:\\.\pipe\LOCALDB#F76AE09E\tsql\query;Integrated Security=True;Connect Timeout=30;Encrypt=False;Trust Server Certificate=False;Application Intent=ReadWrite;Multi Subnet Failover=False"
```

### Docker
#### Secret env variable in Dockerfile
```
# In Dockerfile
RUN --mount=type=secret,id=SECRET_NAME SECRET_NAME="$(cat /run/secrets/SECRET_NAME)" <command_that_uses_secret>

# How to call build (with $SECRET_NAME available as env variable)
docker build . --secret id=SECRET_NAME
```

### Python
- To optimize management of pipenv (keep in local dir to cache dependencies), set env variable `PIPENV_VENV_IN_PROJECT` to `1`
- Look at Astral `ruff` instead of `pylint` (apparently much faster)
- Use Astral [uv](https://github.com/astral-sh/uv) for package and project management - made in Rust - super fast

### AWS Load Balancer (with HTTPS termination)
* Make sure to configure the web server to handle forwarded headers - see [HTTP headers and Application Load Balancers - Elastic Load Balancing (amazon.com)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/x-forwarded-headers.html)
* For ASP.NET Core, see [Configure ASP.NET Core to work with proxy servers and load balancers | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/proxy-load-balancer?view=aspnetcore-7.0)
	* However, see [c# - .net Core X Forwarded Proto not working - Stack Overflow](https://stackoverflow.com/questions/43749236/net-core-x-forwarded-proto-not-working)

### VS Code
#### Mount `.aws` folder
```json
{
	"mounts": [ "source=${env:HOME}${env:USERPROFILE}/.aws,target=/home/vscode/.aws,type=bind"
	]
}
```

### Windows development
- Ensure that all developers have autocrlf turned on:
```bash
git config --global core.autocrlf true  
```
- Create a `.gitattributes` file (see PM Portal)
- Fix line endings:
	- https://git-scm.com/docs/gitattributes#_end_of_line_conversion
	- [Swiss File Knife](https://sourceforge.net/projects/swissfileknife/):  `sfk198.exe lf-to-crlf -yes -dir .`
