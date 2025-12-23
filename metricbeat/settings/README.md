# 🧑🏻‍💻 메트릭비트 설치 가이드
```shell
# homebrew 자체 최신화
$ brew update

# 메트릭비트 설치
$ brew install metricbeat
```

```shell
# 메트릭비트 설치 경로 확인
$ brew info metricbeat
==> metricbeat: stable 9.2.3 (bottled), HEAD
Collect metrics from your systems and services
https://www.elastic.co/beats/metricbeat
Installed
/opt/homebrew/Cellar/metricbeat/9.2.3 (451 files, 143.3MB) *
  Poured from bottle using the formulae.brew.sh API on 2025-12-22 at 07:22:47
From: https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/m/metricbeat.rb
License: Apache-2.0
==> Dependencies
Build: go ✘, mage ✘
==> Options
--HEAD
	Install HEAD version
==> Caveats
To start metricbeat now and restart at login:
  brew services start metricbeat
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/metricbeat/bin/metricbeat
==> Downloading https://formulae.brew.sh/api/formula/metricbeat.json
==> Analytics
install: 32 (30 days), 94 (90 days), 498 (365 days)
install-on-request: 32 (30 days), 94 (90 days), 498 (365 days)
build-error: 0 (30 days)
```
