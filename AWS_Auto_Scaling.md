AWS에서 제공하는 로드 밸런서와 Auto Scaling을 이용한 자동화를 구현해보고자 실습을 진행하고 기록했다.

**인스턴스 생성**
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fci5oIA%2FbtryxJKGs6o%2F4TWkZPzCRME689P5ToU1fK%2Fimg.png)

이미지만 만들면 되기 때문에 기본 설정으로, 보안 그룹은 SSH 만 열어준다 
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbhrAgz%2Fbtryxb10zRY%2Fz5Fz4awHsfkKmxuzbbLlp0%2Fimg.png)

필요한 보안 설정은 추후 추가하면 된다. 일단 모두 열어준 상태로 진행

생성된 인스턴스를 선택하고 작업 > 이미지 및 템플릿 > 인스턴스에서 템플릿 생성을 선택해서 이미지를 만들어준다.

이 이미지를 이용해서 템플릿을 만들어 줄 것이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbC2oER%2FbtryxH0EG7t%2FF0FktEzxEkALPf2RCmL7aK%2Fimg.png)

#### **시작 템플릿 생성**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbIdJis%2FbtryuVFh0O2%2FHzxwjXmnsv02qDrE8NXDl0%2Fimg.png)

템플릿의 이름과 설명을 설정하고 Auto Scaling 지침을 선택해준다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FX9Bo8%2Fbtryu9Ylhm8%2FkonhTqKpeWXVV4605yZmb0%2Fimg.png)

조금 전에 만든 이미지를 선택해준다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbd7mYT%2Fbtryyf3Pn6X%2FzQlWe9gpG70kWknwqlTqd1%2Fimg.png)

프리티어를 선택해주고, 사용할 키 페어를 지정해준다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbwBSex%2FbtryspmBNvY%2F4mDwCp1iYlbjpokEKkJeY0%2Fimg.png)

새롭게 보안 그룹을 생성해준다. 

스크린샷에서는 모든 포트에 대한 HTTP 요청만 열어줬지만, 추가적으로 SSH를 이용하기 위해 내 IP를 통한 SSH 포트도 열어주었다.

이후 오토스케일링 생성으로 이동

#### **오토 스케일링 생성**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fx3NXC%2FbtryvaiGrI8%2FBOFpK1ZFF5Sm7IWkRazWKK%2Fimg.png)

조금 전에 만든 시작 템플릿을 가지고 오토 스케일링을 생성한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcNA2Oc%2Fbtryvvmy6At%2FzxMMGlRs33WKDWxhvySkok%2Fimg.png)

가용 영역은 임의로 4가지 모두 설정해주었다. 
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdfIp5l%2Fbtryt4Xg7nP%2FB6nSvELdFkFEVpoG9hkNJ1%2Fimg.png)

웹서버 이기 때문에 HTTP, HTTPS를 사용하는 Application Load Balancer를 사용.(7 계층)

로드 밸런서는 인터넷에 연결되어 외부에 노출되기 때문에 Internet-facing을 선택한다.

만약 뒷단에서 외부에 노출되지않는 분산 아키텍처를 구축하려면 Internal을 선택하면 된다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcMA9HP%2FbtryvugUzlk%2FrI73iEFNK8wes1rzAUJ3bk%2Fimg.png)

대상 그룹 -> 뒷단의 백엔드를 의미한다.

CloudWatch를 이용해 지표를 수집하는 것이 당연히 좋다. 체크.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbeCm3u%2FbtryxzIjXSQ%2FyfhRDKJOqUVkAOvXlAW8yk%2Fimg.png)

원하는 용량 (Desire)는 보통 최소 용량과 최대 용량의 중간값으로 지정한다.

Scale in이되면 최소 용량으로, out이 되면 최대 용량으로 늘어난다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FU78y3%2FbtryxrKw7hQ%2F6fauN8T812oND5kJKKNU01%2Fimg.png)

조정 정책 -> 동적으로 조정할 것인가? 

동적 조정을 선택하지 않으면 자동으로 되지 않고 우리가 수동으로 스케일을 변경해주는 것.

일반적으로 동적 조정을 선택해서 자동으로 스케일이 지정되도록 한다.

대상 값 -> 우리가 평균 CPU 사용률을 지정했기때문에 Threshold 값이다. CPU의 사용률을 가지고 스케일을 조정하는 설정이며 보통 50~ 75정도로 둔다. 

인스턴스 요구 사항은 우리가 설정해둔 수치에 도달했을 때 어느정도 시간을 기다렸다가 스케일을 조정하는 지에 대한 설정이다. (지연 설정)

만약 0으로 설정하면 수치가 도달하는 순간 바로 스케일을 키우게 된다.

필요 상황에 도달했을 때 바로 스케일을 변경하는게 더 좋아보이긴 하지만, 50 근처에서 수치가 위아래로 변동할 경우 줄였다 키웠다 반복되는 상황이 되면서 성능이 오히려 저하된다.

#### **생성 결과 확인**

생성을 완료하면 인스턴스와 로드밸런서, 대상 그룹 등이 생성된 것을 확인할 수 있다. (가용영역으로 설정했던 4개 영역중 b와 d는 프리티어로 사용할 수 없다며 에러가 뜨고 2개만 생성됐음)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbmhw54%2Fbtryvv7WZcc%2FRj4xvGQ9ID5q1Z3F0okBg1%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcd3kn9%2FbtrytksBoqz%2FL2ocpJKxITcWx2hy5Xvx4K%2Fimg.png)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdOBJhm%2FbtrytgJMLVd%2Fve3s5CDUKthdUQReQlvscK%2Fimg.png)
모니터링에 들가보면 CPU 사용량이 0.3프로에 불과하다

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FZmnw3%2Fbtryxr4TJcu%2Flwmd9aSA9Zi5nSEoQe11KK%2Fimg.png)

이후 오토 스케일이 제대로 작동하는지 보기위해 인스턴스에 부하를 준다.

아래 명령어를 통해 계속되는 부하를 발생시킬 수 있다.

```
[ec2-user@ip-172-31-10-234 ~]$ sha256sum /dev/zero
```

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FkS4z7%2FbtryxISQSQr%2FEzfPRBZAfKM8zWQEDtrel0%2Fimg.png)

시간이 지나고 CPU 사용률이 증가하자

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FR5Rjb%2FbtryyFuzk6v%2FY5qFG72IBtzH8AccWJTNM1%2Fimg.png)

인스턴스가 하나 자동으로 늘어났다 !! (종료된 인스턴스는 아까 이미지를 만들 때 사용하고 지웠던 인스턴스)
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbwILjI%2FbtryyFBlvbD%2FKx680p02QRv1t1OQSyKL8K%2Fimg.png)

EC2 > Auto Scaling > 조금 전 생성한 Auto Scaling 그룹에 들어간 후 

활동 탭에 들어가보면  Auto Scaling의 활동을 볼 수 있는데, 인스턴스가 자동으로 하나 더 생성된 기록을 확인할 수 있다.

시간이 지나면 CPU 사용률이 떨어지지않아서 1개 더 늘어나 초기에 설정한 최대 숫자인 4개가 되는 것을 확인할 수 있다.

참고 : 

Auto Scaling 그룹을 지우지 않고 인스턴스를 먼저 삭제하게 되면 계속해서 내가 설정해둔 인스턴스 개수를 맞추기 위해인스턴스를 만드려고 계속해서 시도하게 된다.