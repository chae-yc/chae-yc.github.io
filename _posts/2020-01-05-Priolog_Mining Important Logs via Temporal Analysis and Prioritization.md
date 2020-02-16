---
title: "논문읽기 - Priolog: Mining Important Logs via Temporal Analysis and Prioritization"
categories:
  - scientist
  - paper_reading
tags:
  - paper
  - log
  - analysis
  - prioritization
last_modified_at: 2020-01-05T19:21:00+09:00
toc: true
published: true
share: false
---

본 글은
**Byungchul Tak & Seorin Park & Prabhakar Kudva, 2019. "Priolog: Mining Important Logs via Temporal Analysis and Prioritization," Sustainability, MDPI, Open Access Journal, vol. 11(22), pages 1-17, November.**
를 읽고 개인적으로 정리한 글입니다.
문제가 발생될 시 포스트를 삭제하도록 하겠습니다.

## Abstract

IT service들 운영을 관리하기 위해서 Log analytics는 중요한 부분이다.
소프트웨어가 복잡해지면서 많은 수의 로그 중에서 problem diagnosis를 위한 의미있는 데이터를 찾아내는 것이 힘들어지고 있다.
본 논문에서는 중요하고 연관성 높은 로그들의 집합으로 만들 수 있는 새로운 기술인 Priolog라는 기술을 소개한다.
Priolog는 log template temporal analysis(시간에 따른 분석), log template frequency analysis, word frequency analysis의 조합을 통해서 분석해서 중요한 로그의 순위를 매겨준다.
이 기술을 구현하여 popular한 OpenStack platform의 problem diagnosis task에 적용했다.
분석 결과에 따르면 Priolog는 여러 시나리오에서 실패 원인에 대한 직접적인 힌트를 담고있는 중요한 로그를 효과적으로 찾을 수 있다.
논문에서 기술의 개념, 디자인 및 실제 로그를 사용한 평가 결과를 보여준다.

> Keywords: log analysis; problem diagnosis; temporal correlation; log template; hierarchical clustering

## 1. Introduction

오늘날 IT 서비스 관리에는 로그 분석 기능이 반드시 필요하다.
로그 메시지는 서비스의 상태 나 문제에 대한 직접적인 힌트로 사용될 수 있다.
containerization [7], micro-service [8,9], serverless computing [10]의 예들 처럼 작은 컴포넌트들을 여러 개 사용하는 것으로 같은 software architecture paradigm이 바뀌고 있다.
각 컴포넌트들이 각자의 log stream을 만들기 때문에 powerful search functions, text pattern matching, aggretation tools를 사용하더라도
로그의 양과 복잡성 때문에 시스템 운영이 어려워진다.
또한 클라우드 소프트웨어의 variety와 diversity로 인해 소프트웨어 및 시스템에서 생성 된 로그는 format, levels of detail, content가 매우 다른 경향이 있다.

로그를 사용한 service failure를 진단하기 위해 이 분야에서 다양한 접근 방식이 제안되어왔다.
대부분의 연구는 failure가 발생한 후 anomalies 또는 outliers를 detect하는 데 중점을 둔다 [11–17].
Log analysis은 data mining 그리고 AI 기술을 적용하기에 좋은 타겟이어왔다 [18].
많은 기술이 이용 가능하지만, 이전의 연구에서는 클라우드 운영 환경에서 실제 중요성과 관련성을 드러내기보다는
post-mortem analysis(사후 분석)을 통해 anomlies detect하거나
모니터링된 데이터의 statistical analysis 만을 제공한다.
따라서, 운영자가 면밀한 검사를 위해서는 관련된 log set로 신속하게 좁혀 나갈 수 있게 만들어야 한다는 과제가 여전히 남아있다.

게다가, 클라우드 환경들은 apllications, system software, patches, cluster provisioning, tenancy(차용: 여러 사용자가 동시에 사용) 그리고 configuration을 변경하는 것이 포함된다.
통계적으로 드물게 발생하지만, anomalies나 outliers가 감지되더라도 실제 복잡한 cloud setting에서는 operational context에 맞춰서 정상적일 수도 있다.
(예: 드물지만 정상적인 load changes, software update and pathces, routine configuration modifications, system utilization changes)
복잡성을 없애기 위해 모니터링 도구 및 분석 기술의 사용이 증가함에 따라 잘못된 경보가 많이 발생하여 시스템 관리자가 파악하기가 힘들다.
또한, 다양한 로그 유형은 개발자에게도 과제를 부여한다.
따라서 clouad operational domain context에서 analytics 또는 AI identified statistical outliers가 필터링되거나 적어도 중요성에 따라 정렬되는 것이 핵심이다.
이상적인 로그 기반 경고 시스템은 outliers를 예측하기 위해 outlier들 끼리의 correlation(상관 관계)를 검토할 뿐만 아니라 active system operation들 간의  temporal correlations(시간적 상관 관계)도 검토해야한다.
(예: planned configuration changes, planned or unplanned maintenance schedules, load patterns, daily health check runs, white lists, etc.).
이 oprational domain의 지식을 AI 및 analysis와 통합하고 연관시키는 것이 이상적인 목표이다.

이를 위해, Priolog를 설계하고 구현했다.
high-level에서 Priolog는 3개의 독립적인 분석을 한다.

- Log template temporal corelation analysis
  - Outlying log message type을 찾기 위해, log message type의 시계열(time-series) 중에서 상관 관계를 검토한다. (시간 별 로그들의 상관 관계를 파악)
    - Log message type 또는 log templete은 contextual 값이나 현재 실행 상태를 반영하는 로그 메시지의 정적 문자열의 일부이다.
  - Raw log stream을 각 log templete에 맞춰 하나씩 n 개의 시계열 값들로 변환한다. 그리고 temporal correlation의 강도에 따라 클러스터한다.
  - 직관적으로, 다른 클러스터와 제대로 클러스터되지 않은 log message type은 중요한 정보를 포함 할 가능성이 있는 abnormal behavior에 의해서 생긴 로그이다.
  - 이러한 log templete들이 log template temporal corelation analysis에서 높은 점수를 얻는다.
- Log template frequency analysis
  - Log message type 별로 log message의 빈도를 보고 정상적인 빈도 수준에서 크게 벗어난 것을 식별한다.
  - 특정 log template에서 빈도가 갑자기 변경되는 경우 비정상적인 활동을 나타내는 것일 수 있다.
  - 이것을 조사하는 것이 근본 원인을 찾는 데 도움이 될 수 있다.
  - 마찬가지로 새로 나타나는 log template types (즉, 빈도가 0에서 일부 값으로 증가)에 중요한 정보가 포함될 수 있으므로 높은 점수를 부여하게 된다.
- Term frequency analysis (TF analysis)
  - Log mesage 내의 개별 단어의 희귀성을 고려하여 log message의 점수를 계산한다. Log message의 점수는 단어 하나가 가지는 희귀 점수를 따로 계산한 결과를 가지고 계산된다.
  - 그 이유는 다른 메시지에서 볼 수 없는 그 특정 단어가 abnoraml한 상태에 대한 직접적인 설명일 수가 있기 때문이다.
  - 이 단계는 이전 두 분석에서 유사한 점수를 가진 log template을 더 좁히기위한 것이다.

최종 출력으로 Priolog는 세 개의 분석의 결과로부터 각 순위의 곱으로 정렬 된 log message type list을 생성한다.

Real-world problems에서 이 방법론의 효과성을 검증하기 위해, Priolog를 OpenStack [19]의 problem determination task에 적용해보았다.
OpenStack은 오픈 소스 IaaS (Infrastructure-as-a-Service) 플랫폼이다.
oversized VM 의 launch, core component의 오류로 인한 VM launch 실패, VM volume attach limit를 초과에 대한 실패 시나리오를 만들었다.
각 실패 사례에 대해, Priolog는 top-ten log message type list를 통해 직접적인 힌트를 포함하는 관련성이 높은 로그를 성공적으로 나열하였다.

Priolog의 목표는 사용자에게 가장 중요한 로그를 먼저 보여주고 문제점 진단 및 근본 원인 분석을 할 수 있게 도와주는 것이다.

- (i) 세 개의 독립적인 분석을 통해 만들어지는 새로운 log 선택 알고리즘
- (ii) 널리 사용되는 소프트웨어를 통해 평가하여 타당성 입증

(i)에서 단일 기준이 가장 중요한 로그를 검색하는 데 효과적이지 않다는 것을 알지만 합리적인 정확도를 얻으려면 여러 기술의 조합을 적용해야 한다.
(ii)에서 application log에는 숨겨진 유용한 정보가 많이 포함되어 있다는 것을 알 수 있다.

아래의 논문 내용은

- Section 2: 아키텍처 디자인과 그 정당성(justifications)
- Section 3: OpenStack에서 Priolog의 효용성을 평가한다.
- Section 4: 관련된 작업들이 기술된다.
- Section 5: 결론

으로 구성되어있다.

## 2. Design of Priolog

Priolog에서 사용되는 input data 들은 아래와 같다.

- Log template list: 이것은 log template discovery algorithm에 의해서 list 이다.
log template discovery를 하는 기술은 out-of-scope이다.
L미 존재하는 기술을 사용해서 이 list를 만드는 것이 가능하다고 가정한다.
Log template lisst는 (a) Log Template Time Series Analysis 에서 사용되고
(b) Log Template Frequency Analysis stage에서도 사용된다.
- Normal logs: 관심 있는 어플리케이션의 normal and error-free 실행 상태에서 수집된 로그들이다.
(b) Log Template Frequency Analysis stage에서 log template frequency vector를 빌드하는데 에만 사용한다.
- Target logs: 몇개의 문제점이 있는 어플리케이션 인스턴스에서 수집된 로그들이다.
보고된 문제들의 root cause를 결정하기 위해서 사용되는 주요 input data이다.

![Figure 1.Priolog Architecture](/assets/images/2020-01-05-Priolog Mining Important Logs via Temporal Analysis and Prioritization/Priolog-Architecture.png)

3개의 analysis는 log template들의 ranked list를 각각 생성한다.
최종 점수는 그 세 개의 analysis에서의 rank들의 곱이다.
penalize(disadvantage 주기)를 위해서 랭크의 합이나 평균 대신 곱을 선택하였다.
합 또는 평균 순위는 변동이 심한 log template과 순위 변동이 없고 일관된 log template을 구별 할 수 없다.

### 2.1 Template Temporal Correlation Analysis

이 분석의 목표는 어플리케이션에서 일어나는 주요 작업과 관계가 없는 log template의 집합을 식별하는 것이다.
그러한 log template를 outlier라고 한다.
이 분석의 뒷 받침은 아래와 같다.

어플리케이션들은 고정되어 있거나 반복되는 순서를 가진 로그들을 생성하는 경향이 있다.
어플리케이션들의 모든 작업들은 두가지 종류로 분류할 수 있다.

- 주기적이고 자동화된 백그라운드 작업
- 다른 컴포넌트나 사용자의 요청에 의해 발생된 predefined set of operation.

특정 작업을 실행 했을 때 여러 개의 로그 메시지가 생성됩니다.
만약 그 로그들을 취합하고 log template들의 수를 센다면, 상호 발생되는 log template들은 어떠한 비율을 유지할 것이다.
로그 파일에서 보는 전체적인 로그들은 이러한 많은 활동의 interleaving이다.

문제 진단을 위해서는 애플리케이션 내에서 일반적인 활동의 일부가 아닌 중요도가 높은 로그를 찾아야한다.
시간적인 상관 관계가 많은 log template를 필터링하고
문제가 발생하였을 때 오류 처리 로직을 실행하면서 발생하는 새롭거나 드문 log template들을 수집한다.

이 원칙을 활용하기 위해 먼저 로그를 각 log template 당 시계열 데이터로 변환한다.
그런 다음 시간적 상관 관계가 높은 로그 템플릿을 식별하여 묶는다. (cluster)
이 작업이 끝날 때 클러스터에 속하지 않은 log template이 있으면 이상치로 처리된다.

#### 2.1.1 Log templates

![log-templates-OpenStack](/assets/images/2020-01-05-Priolog Mining Important Logs via Temporal Analysis and Prioritization/log-templates-OpenStack.png)

Log template들은 실제 로그를 생성하는 것에 사용되는 템플릿으로 사용되는 정적 문자열 패턴(static string pattern)의 유한집합(finite set) 이다.
일반적으로 어플리케이션 코드의 로그 출력 부분에 하드코드 되어 있다.
표 1은 OpenStack 플랫폼에있는 일부 예제 로그 템플릿이다.
Log template 내의 변수 부분은 정규식 표기법에 따라 와일드 카드로 표시되어있다.

비록 표 1에는 10개의 template 밖에 없지만, 전체 크기의 목록은 수백, 수천일 수 도 있다.
log template의 빈도 분포는 멱법칙(power law) 패턴을 보인다.
대부분의 로그 메세지는 전체 lopg template의 작은 부분집합 이라는 것을 암시한다.
또한 드물게 사용되는 많은 수의 log template이 있다.
어플리케이션을 실행하는 중에 비정상적인 상황이 발생할 때까지 일부 log template은 보이지 않는다.

log tempplate list는 기존 기술을 사용하여 특정 어플리케이션에 대해 준비가 되어 있다고 가정한다.
주어진 로그 데이터 집합에서 log template을 정확하게 발견하는 것은 활발한 연구 분야이다[20-23].

#### 2.1.2 Log Template Time-Series Generation

시간적 상관 관계를 분석하기 위해서 첫번째 단계로써 Priolog는 target log를 가지고 log template 목록의 길이가 n이라고 가정하고 n 시계열로 변환한다. 시계열 생성을 위해서, time window `ρ`를 정의하고 각 log template의 각 window에 나타나는 로그 수를 계산한다.
Window의 크기는 편의를 위해 로그 시작 및 종료 시간을 50으로 나누어 결정한다.
이 계수는 필요에 따라 다른 값으로 조정할 수 있다. 이 논문의 경우에는 대략 10 ~ 100ms 이다.

그림2에서 실제 로그로부터 얻은 샘플 시계열 데이터를 볼 수 있다. 3개의 축(log template IDs, time, and log counts)이 있기 때문에, 로그 갯수의 양에 따라 heat-map style을 사용하였다.
작은 정사각형은 로그의 존재를 나타낸다 그리고 색이 강할 수록 관계가 높다.
만약 빨간색이 강하면 그 time window에 많은 수의 로그가 있다는 것을 말한다.
log template ID는 더 자주 사용된 log template이 작은 ID를 가지도록 정렬되었다.
log template ID 0은 분류되지 않는 특이한 template이다.
log template이 몇개의 시간 값에 각자 그룹을 지어 모여있는 것을 확인 할 수 있다.
그림의 오른쪽으로 log template ID가 증가할 수록, log의 수는 줄어들고 time window가 거의 비어있음을 알 수 있다.

![log-templates-heatmap](/assets/images/2020-01-05-Priolog Mining Important Logs via Temporal Analysis and Prioritization/log-templates-heatmap.png)

#### 2.1.3 Correlation Matrix Construction

log template의 갯수로부터 유도되어진 시계열의 쌍들을 얻기 위해서, 상관 계수 (correlation coefficient)를 계산하고 `n x n`의 상관 행렬(correlation matrix)을 만들었다.
Pearson correlation coefficient metrix (which is defined as the covariance of two time series data divided by the product of two standard deviations) 를 적용해서 correlation을 계산하였다.
1에 가까운 값이 높은 상관 관계가 있다는 것을 나타낸다.

다음 계층 클러스터링(hierarchical clustering) 단계에서 log template 시계열 사이의 거리를 초기화하기 위해이 단계를 수행한다. hierarchical clustering이 진행됨에 따라 correlation matrix에는 새로운 시계열이 추가되고 필요에 따라 correlation coefficient가 계산된다. hierarchical clustering 단계는 다음 section에서 자세히 설명한다.

직관적으로, log template time series의 강력한 상관 관계는 동일한 시스템 활동들로부터 만들어진다고 볼 수 있다. 어플리케이션 로직은 결정적이지 않은 작은 변화가 있을 수 있지만 주어진 작업에 대해 동일한 log를 생성하므로 어플리케이션의 특정 작업은 특정 log template 하위 집합(subset) 안에서 로그를 생성하는 경향이 있다. 이 동작은 log template time series를 클러스터링하여 발견하려고하는 시간적으로 상관 된 일련의 log template 수를 생성한다.

#### 2.1.4 Hierarchical Clustering

이전 단계에서 만든 correlation coefficient matrix를 사용하여 hierarchical clustrering을 수행한다.

이전 단계의 초기 상관 계수 매트릭스를 사용하여 Priolog는 이제 계층 적 군집을 수행합니다. 상향식 방식을 사용하여 현재 클러스터 집합 중에서 가장 높은 상관 계수를 갖는 두 개의 로그 템플릿 시계열 τi 및 τj를 반복적으로 클러스터합니다. 그런 다음이 두 로그 템플릿 시계열은 하나의 밀집된 시계열 τk로 병합되어 다시 클러스터링으로 돌아갑니다. 새로운 계수가 상관 계수 매트릭스에 추가됩니다. 동시에 τi와 τj는 매트릭스에서 제거되어 매트릭스의 전체 크기가 두 차원에서 1 씩 줄어 듭니다. Pearson 상관 관계는이 새로운 항목을 행렬에 채우도록 계산됩니다. 그런 다음 나머지 로그 템플릿과 병합 된 그룹간에 다음으로 높은 상관 관계 값을 찾습니다. 이 병합 프로세스는 하나의 최종 클러스터가 될 때까지 반복됩니다. 이러한 프로세스는 알고리즘 1에 설명되어 있습니다. 그림 3a는 덴드로 그램으로 계층 적 클러스터링의 결과를 보여줍니다.

![clustering-log-templates](/assets/images/2020-01-05-Priolog Mining Important Logs via Temporal Analysis and Prioritization/clustering-log-templates.png){: width="800" height="250"}

#### 2.1.5 Outlier Identification

계층 적 군집 구조의 구성은 아직 로그의 중요성을 정량화하지 못합니다. 이를 얻기 위해 가장 오랫동안 병합되지 않은 시계열의 시간을 사용합니다. 로그 템플릿 시계열이 다른 로그 템플릿과 상관 관계가 낮은 경우 클러스터링 알고리즘에 의해 오랫동안 선택되지 않은 상태로 유지되며 클러스터링 프로세스의 끝으로 병합됩니다. 이는 이러한 로그 템플릿이 시스템에서 비정상적으로 비정상적인 활동의 산물임을 의미합니다. 일부 로그가 자연스럽게 정상적인 동작으로 생성되면 다른 여러 발생 로그 템플릿과 더 높은 상관 관계가 나타납니다. 특이 치는 단일 로그 템플릿이거나 클러스터링 초기에 병합되어 오랫동안 병합되지 않은 작은 로그 템플릿 그룹 일 수 있습니다. 우리의 목표는 그러한 로그 템플릿 (그룹)을 찾아 중요하거나 '주의가 필요한'것으로 강조하는 것입니다.
그림 3b는 (a)에서 정렬 된 수직 모서리 목록입니다. 가장 왼쪽에있는 세로선은 그림의 시계열 781에서 가져 왔으며 가장 관련이없는 시계열 (및이 시계열을 구성하는 로그 템플릿)입니다. 시계열은 클러스터링 프로세스 중에 여러 로그 템플릿을 병합하여 생성 된 시계열 일 수 있습니다. 이러한 경우, 해당 로그 템플릿 그룹이 함께 동일한 활동의 일부이지만 그룹으로서 다른 활동과 비교할 때 독창적입니다. 이 정렬 된 수직선 목록을 사용하여 분석 결과로서 로그 템플릿의 최종 순위가 지정됩니다.

### 2.2 Template Frequency Analysis

이 분석의 접근 방식은 관측 된 빈도에 따라 로그 템플릿의 점수를 매기는 것입니다. 이 분석의 기본 가정은 특정 로그 템플릿의 로그 수가 정상보다 높은 빈도를 나타내는 경우이 로그가 중요하다는 것입니다. 이 아이디어는 Sabato et al. [24]. 이 아이디어를 성공적으로 적용하기위한 핵심은 각 로그 템플릿에 대해 '정상'빈도 수준을 정의하는 방법입니다. 우리는 Priolog에서이 방법을 분석 중 하나로 채택합니다.

#### 2.2.1 Frequency Vector Construction

처음 제안되었을 때, Sabato et al. [24] 이기종 작업을 수행하는 많은 컴퓨터가있는 환경을 가정했습니다. 따라서이 작업의 초점 중 하나는 유사한 작업을 수행하는 정확한 컴퓨터 클러스터를 구축하기 위해 클러스터링을 적용하는 것이 었습니다. 그런 다음 '정상 성'은 이러한 군집별로 정의됩니다. 그러나 설정에서 많은 수의 서버가 있다고 가정하지 않습니다. 대신 준비 환경을 프로덕션 설정과 동일하게 유지한다고 가정하고 계획된 실험을 수행하여
‘정상’달리기 그럼에도 불구하고 이것의 결과는 같습니다-확률 벡터 P = {p1, p2, ..., pn}. 확률 벡터 P는 길이 n, 로그 템플릿 수이며, 해당 로그 템플릿이 로그에 나타날 확률을 유지합니다.
목표 로그가 주어지면 Priolog는 확률 벡터와 유사한 주파수 벡터 F를 구성합니다. 이 주파수 벡터는 모든 원소의 합이 1이되도록 P로 정규화되도록 정규화됩니다. 그런 다음 Fi와 Pi의 비율을 | Fi / Pi | 각 로그 템플릿마다 i. 이러한 비율의 로그 템플릿은 순위를 정하는 데 사용됩니다. 로그 템플릿을 이전에 알 수없는 경우 Pi에 있지 않습니다. 이 경우 새로 표시되는 로그가 중요 할 수 있으므로 비율은 무한대이고 해당 로그 템플릿의 순위는 1입니다.

### 2.3 Term Frequency (TF) Analysis

처음 두 분석은 중요하지 않은 로그를 배제하여 로그 볼륨을 줄이는 데 효과적입니다. 불행히도 이러한 이유로 인해 중요한 정보가 포함 된 로그를 정확하게 찾아내는 데 적합하지 않습니다. 템플릿 시간 상관 분석 (TTCA)의 경우이 분석이 제대로 작동하려면 시계열 데이터에 데이터가 풍부해야합니다. 그러나 인기없는 많은 로그 템플릿의 경우 시계열은 대부분 0으로 채워지고 때로는 0이 아닌 경우가 많습니다. 이러한 시계열은 의미있는 상관 계수를 생성하지 않습니다. 이것은 시간 정보에 의한 클러스터링이 우리에게 흥미가없는 클러스터 만 제거 할 수 있음을 의미합니다. 대부분의 경우 가치가 높은 로그는 일반적으로 거의 발생하지 않는 로그 템플릿에서 가져옵니다. 템플릿 빈도 분석 (TFA1)의 경우, 이에 대한 문제점은 로그 템플릿 빈도의 증가가 이러한 로그가 더 중요하다는 것을 강력하게 나타내지 않는다는 것입니다. 오히려 증가 된 로그 인스턴스는 일반적이기 때문에 덜 가치있는 것으로 간주 될 수 있습니다. 또 다른 문제는 정상적인 경우에는 존재하지 않는 갑자기 나타나는 로그 템플릿이 너무 많을 수 있다는 것입니다. 이 문제가 자주 발생하여 로그를 가장 관련성이 높은 로그로 좁힐 수 없었습니다.
다른 분석의 단점을 해결하기 위해 세 번째 주파수 분석 (TFA2)이 도입되었습니다. 가설은 후보 로그 중 하나에 다른 로그 템플릿에 자주 표시되지 않는 단어가 포함 된 경우 해당 단어가 중요하다는 가설입니다.

프로그래머가 현재 상태가 표시되는 방식으로 코드에 로그 인쇄 명령문을 추가했을 수 있으므로 비정상 상태를 직접 설명하는 단어 일 수 있습니다. 이 아이디어를 바탕으로 세 번째 분석으로 단어 수준의 빈도 분석을 수행합니다.
로그 템플릿에는 숫자, 특수 문자 및 문장 부호 문자를 포함하는 많은 토큰이 포함되어 있습니다. 이 분석에서 우리는 알파벳과 밑줄을 포함하는 토큰만을 의미합니다. 순위 ri의 점수는 식 (2)로 계산됩니다. Si를 사용하여 i 번째 로그 템플릿에 설정된 총 단어를 나타냅니다. i 번째 로그 템플리트에서 유효 단어의 총 수는 | Si |입니다. fw를 전체 로그 템플릿 세트에서 단어 w의 빈도로 둡니다. 그런 다음 fw / ∑w 'fw'는 로그 템플릿 세트에서 전체 수준 w의 전체 비율이됩니다. 즉, 주어진 로그 템플릿 내 모든 단어의 비율 평균을 계산하여 점수로 사용합니다.

1 fwj
ri = ∑ () (2)
   | Si | j∈Si ∑w ′ fw ′
이 분석의 최종 순위 목록은이 ri 점수로 로그 템플릿을 정렬하여 생성됩니다.

## 의문점

1. 읽는데 왜 이렇게 오래 걸릴까
