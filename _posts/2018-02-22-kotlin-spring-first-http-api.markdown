---
layout: post
title: "Famphlet 프로젝트 버전 관리하기"
date: "2018-02-22 21:02:09 +0900"
category: kotlin
comments: true
tags: Kotlin 스프링 Spring 스프링부트 SpringBoot Git Github 

---

프로젝트를 본격적으로 다뤄보기 전에 버전 관리를 적용하도록 하겠습니다. 블로그도 Github을 이용하고 있는 만큼 Git을 이용해 버전 관리를 하면서 Github과 연동해보죠.

# Git

프로젝트를 진행하기 전에 이전 포스트까지 만들었던 소스를 깃헙에 올리도록 하겠습니다. 아무래도 버전관리를 해야 삽질을 해도 기댈 곳이 있겠죠?

우리는 인텔리제이를 사용하고 있기 때문에 인텔리제이에서 손쉽게 [Git](https://git-scm.com/)으로 버전관리를 할 수 있습니다. Famphlet 프로젝트가 열려 있는 상태에서 [VCS > Enable Version Control Integration...](https://www.jetbrains.com/help/idea/enable-version-control-integration-dialog.html) 메뉴로 진입하세요.

![깃 저장소 만들기 메뉴]({{"/assets/image/kotlin-famphlet/first-controller/create-git.png" | absolute_url}})

메뉴로 진입하면 버전 관리 시스템을 선택하는 화면이 뜹니다. 여기서 Git을 고르면 이후에 깃헙과 연동할 수 있습니다.

![깃 저장소로 지정할 디렉토리 선택하기]({{"/assets/image/kotlin-famphlet/first-controller/create-git-selector.png" | absolute_url}})

이제 Famphlet 프로젝트는 버전 관리 대상이 되었습니다. 그러나 바로 모든 파일들의 버전이 관리되는 것이 아니기 때문에, 관리할 대상을 Git에 추가해 주어야 합니다. 아마 지금은 파일들이 붉은 색으로 보일겁니다.

![버전 관리 되지 않는 파일들의 모습]({{"/assets/image/kotlin-famphlet/first-controller/unversioned-files.png" | absolute_url}})

버전 관리가 필요한 파일들만 선택하여 Git에 추가합시다. ([git add](https://git-scm.com/docs/git-add) 명령) Famphlet 프로젝트에서는 `src` 디렉토리와 하위의 모든 파일, 그리고 `build.gradle`과 `settings.gradle`정도가 되겠습니다. 해당 파일들을 선택하고 오른쪽 버튼을 눌러 [Git > Add](https://www.jetbrains.com/help/idea/adding-files-to-version-control.html) 메뉴를 실행합시다. (Ctrl + Alt + A, Command + Alt + A)

![버전 관리가 필요한 대상 파일을 깃에 추가하기]({{"/assets/image/kotlin-famphlet/first-controller/add-changes.png" | absolute_url}})

이렇게 추가된 파일들은 프로젝트 도구 창에서도 초록색으로 보이게 됩니다.

![버전 관리 되는 파일들의 모습]({{"/assets/image/kotlin-famphlet/first-controller/versioned-files.png" | absolute_url}})

이제 커밋해 봅시다.([git commit](https://git-scm.com/docs/git-commit) 명령) 커밋은 변경된 파일을 하나의 버전으로 기록하는 것입니다. 기록은 자주 일어나므로 1.0, 1.1 이런 식으로 관리되지 않습니다. 그 버전이 어떤 버전인지 알기 위해 자연어로 기록합니다. 커밋을 하기 위해서 [VCS > Git > Commit Directory...](https://www.jetbrains.com/help/idea/checking-in-files.html) 메뉴로 진입하세요.

![커밋 메뉴]({{"/assets/image/kotlin-famphlet/first-controller/commit-directory-menu.png" | absolute_url}})

혹은 오른쪽 위의 네비게이션 도구 막대에서 커밋 아이콘을 누르거나

![네비게이션 도구 막대의 커밋 메뉴]({{"/assets/image/kotlin-famphlet/first-controller/commit-menu-top.png" | absolute_url}})

VCS 도구 창에서 커밋 아이콘을 누를 수도 있습니다.

![VCS 도구 창의 커밋 메뉴]({{"/assets/image/kotlin-famphlet/first-controller/commit-menu-vcs-tool-window.png" | absolute_url}})

무엇보다 가장 편한 것은 단축키를 이용하는 방법입니다. (Ctrl + K, Command + K)

커밋 메뉴로 진입하면 [커밋 대화상자](https://www.jetbrains.com/help/idea/commit-changes-dialog.html)가 뜹니다. 대화상자에는 버전 관리 대상인 파일들이 초록색으로 표시되며 체크가 되어 있습니다. 그리고 제일 밑에 Unversioned files라는 항목이 있을겁니다. 여기엔 우리가 `Git > Add` 메뉴로 추가하지 않은 모든 파일이 있습니다. 이것들은 버전 관리를 안할 생각이므로 체크가 해제된 상태로 두겠습니다.

자 파일 목록 밑에 `Commit Message`라는 칸이 있습니다. ([git commit -m](https://git-scm.com/docs/git-commit#git-commit--mltmsggt) 명령) 이곳이 바로 이 버전의 설명을 적는 공간입니다. 위 목록들은 Git에 처음으로 기록되는 버전이므로 이 버전의 커밋 메시지는 다음과 같이 하겠습니다.

> Famphlet 프로젝트에서 '안녕, 여러분'으로 시작하기

이제 Commit을 눌러봅시다. 앗!? [Code Analysis](https://www.jetbrains.com/help/idea/code-analysis.html) 대화상자가 뜨네요. 에러는 없지만 2개의 경고가 있다는데, Review 버튼을 누르면 어떤 경고인지 확인할 수 있습니다.

![코드 분석 대화상자]({{"/assets/image/kotlin-famphlet/first-controller/code-analysis-dialog.png" | absolute_url}})

두 개의 경고 모두 `Warning:(1, 1) Package directive doesn't match file location`이네요. 아니, 코틀린은 분명 패키지와 디렉토리 구조가 달라도 상관이 없다고 했는데 무슨 일일까요? `File>Settings...` 메뉴로 설정 창을 띄워보겠습니다. 인텔리제이는 설정이 매우 많기 때문에 설정을 찾을 수 있는 검색 막대가 있습니다. 매우 유용하게 쓰고 있습니다.

검색 막대에 `package directive`라고 입력하니 Editor의 Inspections 설정이 나타납니다. 그 외에도 몇몇 설정이 있긴 하지만 코드 분석 결과를 보고 찾아본 것이니 Inspections 설정이 맞겠지요. 오른쪽을 보면 Kotlin 하위로 Java interop issues 중의 하나로 되어 있네요. 하지만 Famphlet은 이제 더이상 Java와 혼용할 생각이 없으므로 이 검사는 하지 않아도 됩니다. 체크를 해제하여 이 검사는 무시하도록 하겠습니다.

![코드 검사기 설정 화면]({{"/assets/image/kotlin-famphlet/first-controller/inspections-settings.png" | absolute_url}})

자 이제 다시 커밋을 해봅시다. 거침 없이 커밋이 되는군요. 커밋이 완료되었다는 것을 Version Control 도구 표시 막대에 표시해 주는 친절함도 보여줍니다.

![커밋 완료 말풍선]({{"/assets/image/kotlin-famphlet/first-controller/committed-tooltip.png" | absolute_url}})

그런데 커밋 화면에서 봤던 `Unversioned files`가 마음에 걸립니다. 이녀석들은 제가 커밋을 할 때마다 변경된 파일들 사이에서 제 초점을 흐리겠지요. 저는 앞으로도 이들의 버전 관리를 할 생각이 없으므로 제 눈 앞에서 사라지게 만들 요량입니다. Git은 버전 관리가 필요없는 파일들을 버전 관리 시스템에서 완벽하게 배제하기 위한 방법으로 `.gitignore`라는 파일에서 그 목록을 관리할 수 있게 해줍니다. 이 파일을 프로젝트 최상위 경로에 만들어 봅시다. 만들면 아마 깃에 추가할지를 묻는 대화상자가 나타날겁니다. 다른 곳에서 이 프로젝트를 `Checkout`하여 작업할 때도 버전 관리하고 싶지 않은 파일들이므로 `.gitignore` 파일은 Git 저장소에 함께 있는 것이 좋겠습니다. 이 대화상자에서는 Yes를 선택합니다.

![파일을 깃에 추가할지 선택하는 대화상자]({{"/assets/image/kotlin-famphlet/first-controller/add-file-to-git.png" | absolute_url}})

파일이 추가되었습니다. 파일 목록을 보시면 빨간색이 아니라 초록색일겁니다. 방금 대화상자에서 Yes를 선택했기 때문이죠. 이제 이 파일에 사라졌으면 싶던 그 녀석들을 입력해보겠습니다.

{% highlight gitignore %}
.gradle
.idea
build
out
{% endhighlight %}

이제 커밋을 해보면 `Unversioned files`가 안보일 겁니다. 속이 다 시원하네요.

![깃 저장소로 지정할 디렉토리 선택하기]({{"/assets/image/kotlin-famphlet/first-controller/commit-gitignore.png" | absolute_url}})

인텔리제이의 [.ignore 플러그인](https://plugins.jetbrains.com/plugin/7495--ignore)을 설치하면 무시하고 싶은 파일을 쉽게 목록에 추가할 수 있습니다. 그저 오른쪽 버튼을 눌러서 메뉴를 클릭만 하면 됩니다.

![플러그인 설정 화면]({{"/assets/image/kotlin-famphlet/first-controller/plugin-settings.png" | absolute_url}})

![이그노어 플러그인 설치 화면]({{"/assets/image/kotlin-famphlet/first-controller/ignore-plugin.png" | absolute_url}})

![대상 파일에서 오른쪽 버튼 눌러 쉽게 버전 관리 무시하기]({{"/assets/image/kotlin-famphlet/first-controller/ignore-context-menu.png" | absolute_url}})

# Github

이제 Famphlet 프로젝트의 소스 파일 버전을 관리할 수 있게 되었습니다. 하지만 저장장치가 먹통이 된다면? 프로젝트 소스도 사라져 버릴테지요. [Github](https://github.com/)과 연동하면 프로젝트 디렉토리가 통째로 날아가도 다시 프로젝트 소스를 되찾을 수 있습니다. 노트북에서 작업하고 싶을 때 얼마든지 노트북에서 프로젝트 소스 파일을 획득할 수 있습니다. 먼저 Github 저장소를 만드는 일부터 시작해보죠. Github에 방문하여 로그인을 하면 초록색 [New repository](https://github.com/new)버튼을 보실 수 있습니다.

![깃헙 메인 화면]({{"/assets/image/kotlin-famphlet/first-controller/github-main-page.png" | absolute_url}})

새로운 저장소를 만드는 것은 매우 간단합니다.
- 저장소 이름을 정하고
- 공개 여부를 설정한 후
- 초록색 `Create repository` 버튼을 누르세요.

물론

- 저장소의 설명을 추가한다거나
- `.gitignore` 파일 또는 `README` 파일을 자동으로 생성할지를 정한다거나
- 라이센스를 고른다거나

하는 부가적인 작업도 할 수 있습니다.

다 만들면 생성된 저장소 화면으로 이동합니다. 화면에서 `Clone or download` 버튼을 누르면 저장소 주소를 복수할 수 있습니다. 여기서 저장소 주소를 복사합니다.

![깃헙 저장소 화면의 저장소 주소 복사하기]({{"/assets/image/kotlin-famphlet/first-controller/clone-or-download.png" | absolute_url}})

이제 인텔리제이로 돌아옵시다. [VCS > Git > Push...](https://www.jetbrains.com/help/idea/commit-and-push-changes.html#push) 메뉴를 누르시면 지금까지 커밋한 버전들을 원격 저장소(여기선 Github)로 밀어 넣을 수 있습니다. (Ctrl + Shift + K, Command + Shift + K)

![커밋한 버전 푸시하기]({{"/assets/image/kotlin-famphlet/first-controller/push-commit-dialog.png" | absolute_url}})

푸시 대화창을 보시면 `Define remote`라고 써있는 것을 볼 수 있습니다. 이곳을 눌러서 복사했던 저장소 주소를 붙여넣읍시다. ([git remote](https://git-scm.com/docs/git-remote) 명령)

![원격 저장소 정의하기]({{"/assets/image/kotlin-famphlet/first-controller/define-remote-dialog.png" | absolute_url}})

저장소 주소를 넣고 나면 `Define remote`로 표시되던 곳이 `origin : master`라고 변경될 것입니다. 이제 푸시 버튼을 누르기만 하면 됩니다. 중간에 로그인 창이 나옵니다만 깃헙 로그인 정보를 넣으시면 됩니다. ([git push](https://git-scm.com/docs/git-push) 명령)

원래는 푸시 과정에 아무 문제가 없어야 하지만 저는 저장소를 만들 때 라이센스를 고르는 바람에 버전 충돌이 있었습니다. 혹시 저와 비슷한 상황을 만나셨다면 인텔리제이 오른쪽 아래에 푸시가 거부되었다는(Rejected) 말풍선이 떠있을 것입니다. 내 컴퓨터의 깃 버전과 원격 저장소의 깃 버전이 충돌한 건데, 아래와 같은 상황이 하나의 예라고 할 수 있습니다.

|내 컴퓨터(로컬)|버전|Github|버전|
|:-|:-|
|Famphlet 프로젝트에서 '안녕, 여러분'으로 시작하기|1|Initial commit|1|
|.gitignore 추가|2|||

버전을 숫자로 쓰긴 했는데 원래는 (b27ede5같은)해시값으로 표시됩니다. 위 표에서 볼 수 있는 것처럼 저는 제 컴퓨터에서 이미 커밋을 두개나 해놓은 상태입니다. 그런데 깃헙에서 저장소를 만들면서 라이센스를 선택하였고, 저장소가 만들어지면서 라이센스(텍스트 파일)가 커밋이 되면서 깃헙 저장소에는 커밋이 하나가 된 것이지요.

내 컴퓨터와 깃헙 저장소의 버전은 일관성이 있어야 합니다. 하나의 프로젝트이고 한 프로젝트의 소스 코드이니 당연하겠지요?

버전 충돌로 푸시가 되지 않을 때는 일단 풀(내려받기)을 해주어야 합니다. 오른쪽 상단 툴바의 [Update Poject](https://www.jetbrains.com/help/idea/sync-with-a-remote-repository.html#pull) 버튼을 눌러서 풀을 해봅시다. ([git pull](https://git-scm.com/docs/git-pull) 명령)

![네비게이션 도구 막대의 풀 메뉴]({{"/assets/image/kotlin-famphlet/first-controller/pull-toolbar.png" | absolute_url}})

누르면 대화 상자가 나옵니다. Update Type을 Rebase로 선택하고 진행해보죠.

![네비게이션 도구 막대의 풀 메뉴]({{"/assets/image/kotlin-famphlet/first-controller/update-project.png" | absolute_url}})

풀을 진행하면 버전이 정리가 됩니다. 이후에 푸쉬를 진행하시면 됩니다. Rebase된 버전들을 보면 이전 버전이지만 커밋 시간이 미래임을 보실 수 있습니다.

![네비게이션 도구 막대의 풀 메뉴]({{"/assets/image/kotlin-famphlet/first-controller/after-rebase.png" | absolute_url}})

Rebase까지 해서 프로젝트를 Github과 연동해보았습니다. 이번 시간에 제대로 된 첫 기능을 붙여보려 했는데 버전 관리를 하면서 보니 길어질 것 같아 잘라갑니다. 다음 시간에는 정말 첫 기능을 붙여 보도록 하겠습니다.
