# 이펙티브자바 2장 - 객체 생성과 파괴

## 아이템2 생성자에 매개변수가 많다면 빌더를 고려하라

* 선택적 매개변수가 많을 경우, static factory와 생성자에는 한계가 있다.   
멤버 변수 중 대다수가 0인 경우를 생각해보자. 
    + 점층적 생성자 패턴

    ```java
    public class NutritionFacts {
        private final int servingSize;  // 1회 제공량(필수)
        private final int servings;     // 총 n회 제공량 (필수)
        private final int calories;     // 1회 제공량당 (선택)
        private final int fat;          // 1회 제공량당 (선택)
        private final int sodium;       // 1회 제공량당 (선택)
        private final int carbohydrate; // 1회 제공량당 (선택) 
    }

    public NutritionFacts(int servingSize, int servings){
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories,0){
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat){
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium){
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate){
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
    //아래는 호출 방식
    NutritionFacts cocaCola = new NutritionFacts(240,8,100,0,35,27);
    ```   
    *매개변수가 '겨우' 6개 뿐이라 그리 나쁘지 않아 보일 수도 있지만, 더 늘어난다면?*   
       
       
    + 자바빈즈 패턴

    ```java
    public class NutritionFacts {
        private int servingSize  = -1;  //필수  
        private int servings     = -1;     //필수
        private int calories     = 0;     
        private int fat          = 0;          
        private int sodium       = 0;       
        private int carbohydrate = 0; 

        public NutritionFacts() {}
        
        //Setters
        public void setServingSize(int val)  { servingSize = val; }
        public void setServings(int val)     { servings = val; }
        public void setCalories(int val)     { calories = val; }
        public void setFat(int val)          { fat = val; }
        public void setSodium(int val)       { sodium = val; }
        public void setCarbohydrate(int val) { carbohydrate = val; }
    }
    //아래는 호출 방식
    NutritionFacts cocaCola = new NutritionFacts();
    cocaCola.setServingSize(240);
    cocaCola.setServings(8);
    cocaCola.setCalories(100);
    cocaCola.setSodium(35);
    cocaCola.setCarbohydrate(27);
    ```

    점층적 패턴보다는 가독성이 올라갔다.
    *그러나, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태! 클래스를 불변(immutable)[아이템17]으로 만들 수 없으며 스레드 안전성을 보장하기 위해서는 개발자의 추가 작업이 필요(아직 이해가 잘 안간다)*

    + 빌더 패턴

    ```java
    public class NutritionFacts {
        private final int servingSize;  
        private final int servings;     
        private final int calories;     
        private final int fat;          
        private final int sodium;       
        private final int carbohydrate; 

        public static class Builder {
            private final int servingSize;
            private final int servings;

            private int calories     = 0;
            private int fat          = 0;
            private int sodium       = 0;
            private int carbohydrate = 0;

            public Builder(int servingSize, int servings) {
                this.servingSize = servingSize;
                this.servings    = servings;
            }

            public Builder colories(int val)       { calories = val; return this; }
            public Builder fat(int val)            { fat = val; return this; }
            public Builder sodium(int val)         { sodium = val; return this; }
            public Builder carbohydrate(int val)   { carbohydrate = val; return this; }

            public NutritionFacts build() {
                return new NutritionFacts(this);
            }
        }

        private NutritionFacts(Builder builder) {
            servingSize  = builder.servingSize;
            servings     = builder.servings;
            calories     = builder.calories;
            fat          = builder.fat;
            sodium       = builder.sodium;
            carbohydrate = builder.carbohydrate;
        }
    }

    //호출 방식
    NutritionFacts cocaCola = new NutritionFacts.Builder(240,8).calories(100).sodium(35).carbohydrate(27).build();
    ```

    위의 빌더 패턴은 특히 "계층적으로 설계된 클래스"와 잘 어울린다. 아래의 피자 예시를 보자
    ```java
    public abstract class Pizza {
        public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
        final Set<Topping> toppings;

        abstract static class Builder<T extends Builder<T>> {
            EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
            public T addTopping(Topping topping) {
                toppings.add(Objects.requireNonNull(topping));
                return self();
            }
        }

        abstract Pizza build();
        protected abstract T self(); // subclass가 override하여 this를 반환하도록

        Pizza(Builder<?> builder) {
            toppings = builder.toppings.clone();
        }  
    }
    ```

    *Pizza.Builder 클래스는 `재귀적 타입 한정(아이템30)`을 이용하는 제네릭 타입*   
    *self 타입이 없는 자바를 위한 `simulated self-type` 관용구 활용 -> 형변환하지 않고도 메서드체이닝 가능*

    ```java
    public class NyPizza extends Pizza {
        public enum Size { SMALL, MEDIUM, LARGE }
        private final Size size;

        public static class Builder extends Pizza.Builder<Builder> {
            private final Size size;

            public Builder(Size size) {
                this.size = Objects.requireNonNull(size);
            }

            @Override public NyPizza build() {
                return new NyPizza(this);
            }

            @Override protected Builder self() { return this; }
        }

        private NyPizza(Builder builder){
            super(builder);
            size = builder.size;
        }
    }

    public class Calzone extends Pizza {
        private final boolean sauceInside;

        public static class Builder extends Pizza.Builder<Builder> {
            private final boolean sauceInside = false; // default value

            public Builder sauceInside() {
                sauceInside = true;
                return this;
            }

            @Override public Calzone build(){
                return new Calzone(this);
            }

            @Override protected Builder self() { return this; }
        }

        private Calzone(Builder builder){
            super(builder);
            sauceInside = builder.sauceInside;
        }
    }
    ```
    *빌더패턴은 상당히 유연하다. API는 시간이 지날수록 매개변수가 늘어나는 경향이 있다는 점을 생각하면 애초에 빌더패턴으로 시작하는게 나은 경우가 더 많다.`Lombok`에서도 빌더패턴을 지원하니, 나중에 따로 공부해보자.*

    


