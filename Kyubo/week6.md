# 7장: 중첩 제거 테크닉
## 7.1 이미 존재하는 기능은 다시 구현하지 말자
- 혹시 표준 컬렉션 라이브러리에 이미 존재하는 메소드가 있는지 잘 찾아보고 구현 하자
- 바퀴의 재발명: 이미 똑똑한 사람들이 만들어놓은걸 굳이 또 만드는건 실력의 미숙함 또는 실수등으로 버그 발생확률이 높아짐

## 7.2 반복처리 내부의 조건 분기 중첩
- 조기 컨티뉴로 조건 분기의 중첩을 제거할수 있다.
  - 조건분기가 중첩되어 있는경우 컨티뉴를 통해 다음 반복으로 넘어가는 로직을 활용하자
- 조기 브레이크로도 중첩을 제거할수 있다.
  - 6장에서 봤듯이 조건 중첩시 조기 브레이크를 통해 가독성을 높일수 있는 코드를 작성할수 있다.

## 7.3 응집도가 낮은 컬렉션 처리
- 컬렉션과 관련된 응집도가 낮아지는 문제는 일급 컬렉션 패턴을 사용해서 해결할 수 있다.
    - 일급 컬렉션 패턴: 컬렉션을 포함하는 클래스를 만들어서 컬렉션을 감싸는 방식으로 컬렉션을 처리하는 로직을 클라스로 분리하는 방식
- 클라스에는 2가지가 있어야됨
    - 인스턴스 변수
    - 인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정상적으로 조작하는 메소드
- 일급 컬렉션은 다음 요소로 구성되어야함
    - 컬렉션 자료형의 인스턴스 변수
    - 컬렉션 자료형의 인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정상적으로 조작하는 메소드
- 외부로 전달할 때 컬렉션 변경 막기
    - 인스턴스 변수를 그대로 외부에 전달하면 외부에서도 맘대로 조작이 가능함
    - 이를 막기 위해 컬렉션의 요소를 변경하지 못하게 막아두는것이 좋음

```swift
// 일급 컬렉션 패턴을 적용한 클라스
class TaskList: ObservableObject {
    @Published var tasks: [String]
    
    init(tasks: [String] = []) {
        self.tasks = tasks
    }
    
    // 컬렉션의 요소를 안전하게 조작할 수 있는 메소드
    // guard let을 통해 빈값이 들어오는것을 방지
    func addTask(_ task: String) {
        guard !task.isEmpty else { return }
        tasks.append(task)
    }
    
    func removeTask(_ task: String) {
        guard let index = tasks.firstIndex(of: task) else { return }
        tasks.remove(at: index)
    }
    
    // 외부에서 변경하지 못하도록 읽기 전용으로 만듬
    func allTasks() -> [String] {
        return tasks
    }
}
```

```swift
struct ContentView: View {
    // 일급 컬렉션을 사용하여 상태 관리
    @StateObject private var taskList = TaskList()

    @State private var newTask: String = ""
    
    var body: some View {
        VStack {
            HStack {
                TextField("새로운 작업 추가ㄱㄱ", text: $newTask)
                    
                Button(action: {
                    taskList.addTask(newTask)
                    newTask = ""
                }) {
                    Text("작업 추가하기")
                }
            }
            .padding()
            
            List {
                // 일급 컬렉션에 만들어놓은 읽기전용 함수로 접근
                ForEach(taskList.allTasks(), id: \.self) { task in
                    Text(task)
                }
            }
        }
    }
}
```