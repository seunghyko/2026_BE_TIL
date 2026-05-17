# 📘 Today I Learned

### 1. 오늘 배운 내용
-절차 지향 프로그래밍과 객체 지향 프로그래밍의 차이점. <br>
-객체의 구성 요소인 상태(변수)와 기능(메서드).  
-메서드 오버로딩(Overloading)의 개념.<br>
-생성자(Constructor)를 활용한 객체 초기화.<br>
-private 접근 제어자, Getter/Setter, 그리고 this 키워드의 사용법.

### 2. 핵심 정리 (내 언어로)
-절차 지향 vs 객체 지향: 절차 지향은 실행 순서와 "어떻게" 동작하는지에 초점을 맞추는 반면, 객체 지향은 객체 안 상태와 메서드를 활용하여 필요할 때 목적에 맞게 사용 <br>



### 3. 실습 / 과제 / 결과물
<pre>
public class MusicPlayer {
    private int volume = 0;
    private boolean isOn = false;

    void on(){
        this.isOn = true;
        System.out.println("음악 플레이어를 시작합니다.");
    }

    void off(){
        this.isOn = false;
        System.out.println("음악 플레이어를 종료합니다.");
    }

    void volumeUp(){
        this.volume++;
        System.out.println("음악 플레이어 볼륨: "+volume);
    }

    void volumeDown(){
        this.volume--;
        System.out.println("음악 플레이어 볼륨: "+volume);
    }

    void showStatus(){
        System.out.println("음악 플레이어 상태 확인");
        if(this.isOn){
            System.out.println("음악 플레이어 ON, 볼륨: "+ this.volume);
        }else{
            System.out.println("음악 플레이어 OFF");
        }
    }

    void setVolume(int volume){
        this.volume=volume;
        System.out.println("음악 플레이어 볼륨: "+ this.volume);
    }
}
```
</pre>

### 4. 느낀 점 & 다음 계획
-절차 지향으로 코드를 작성했을 때보다 클래스를 분리하고 캡슐화를 적용하니 코드의 관리가 크게 향상됨을 느꼈다.