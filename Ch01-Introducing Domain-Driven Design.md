# Chapter 01. Introducing Domain-Driven Design

## 학습 목표

1. **도메인 주도 설계(DDD)의 기본 원칙과 목표**
2. **공유 모델의 중요성과 필요성**
3. **비즈니스 이벤트 중심의 도메인 이해 방법**
4. **도메인을 하위 도메인으로 분할하는 방법**
5. **바운디드 컨텍스트를 통한 해결책 설계**
6. **유비쿼터스 언어의 개념과 중요성**
7. **DDD가 데이터베이스 중심 설계나 객체 지향 설계와 어떻게 다른지**

## 주요 용어 및 정의

### 핵심 개념
- **도메인(Domain)**: 도메인 전문가가 전문성을 가진 영역
- **도메인 모델(Domain Model)**: 특정 문제와 관련된 도메인의 측면을 단순화해 표현한 것
- **하위 도메인(Subdomain)**: 더 큰 도메인의 일부이면서도 고유한 전문 지식을 지닌 영역
- **바운디드 컨텍스트(Bounded Context)**: 해결 공간에서 다른 서브시스템과 구분되는 명확한 경계를 가진 서브시스템
- **유비쿼터스 언어(Ubiquitous Language)**: 도메인과 관련된 개념과 용어의 집합으로, 팀 구성원들과 소스코드 모두가 공유

### 프로세스 관련
- **도메인 이벤트(Domain Event)**: 시스템에서 발생한 사건을 기록한 것 (항상 과거 시제)
- **명령(Command)**: 어떤 프로세스가 실행되기를 요청하는 것
- **이벤트 스토밍(Event Storming)**: 비즈니스 이벤트와 그에 연관된 워크플로를 공동으로 탐색하는 협업 프로세스

### 설계 관련
- **컨텍스트 맵(Context Map)**: 여러 바운디드 컨텍스트와 그 관계를 보여주는 고수준 다이어그램
- **핵심 도메인(Core Domain)**: 비즈니스 경쟁력을 제공하고 수익을 창출하는 영역
- **지원 도메인(Supportive Domain)**: 필요하지만 핵심은 아닌 도메인
- **범용 도메인(Generic Domain)**: 해당 비즈니스에만 고유하지 않은 도메인

## 핵심 내용

### 1. 공유 모델의 중요성
- 개발자와 도메인 전문가가 동일한 모델을 공유해야 함
- "번역" 과정에서 발생하는 왜곡과 손실을 방지
- 코드가 곧바로 공유된 정신적 모델을 반영하도록 설계

### 2. DDD의 4가지 핵심 지침
1. **데이터 구조보다는 비즈니스 이벤트와 워크플로에 집중**
2. **문제 도메인을 더 작은 하위 도메인들로 분할**
3. **해결책 안에 각 하위 도메인의 모델을 생성**
4. **프로젝트 참여자 모두가 공유하는 유비쿼터스 언어 개발**

### 3. 이벤트 스토밍의 가치
- 비즈니스에 대한 공유 모델 구축
- 모든 팀에 대한 인식 확보
- 요구사항의 누락 발견
- 팀 간의 연결 관계 파악
- 리포팅 요구사항에 대한 인식

## FAQ (자주 묻는 질문)

### Q1: 도메인 주도 설계(DDD)란 무엇인가요?
**A:** 도메인 주도 설계는 개발자와 도메인 전문가가 같은 언어로 소통하고, 같은 모델을 공유하면서 소프트웨어를 만드는 방법론입니다. 

예를 들어, 전자상거래 시스템에서 "주문(Order)"이라는 개념을 다룰 때, 개발자가 구현하는 `Order` 클래스와 비즈니스 담당자가 생각하는 "주문"이 정확히 일치해야 합니다. 

```csharp
// DDD 적용 전: 기술 중심의 모델
public class OrderEntity
{
    public long Id { get; set; }
    public string Status { get; set; }
    public DateTime CreatedDate { get; set; }
    // 기술적 필드들...
}

// DDD 적용 후: 도메인 중심의 모델
public class Order
{
    public OrderId Id { get; private set; }
    public OrderStatus Status { get; private set; }
    public Money TotalAmount { get; private set; }
    public List<OrderLineItem> LineItems { get; private set; }
    // 비즈니스 의미가 명확한 필드들...
}
```

이렇게 하면 코드가 실제 비즈니스 요구사항을 정확히 반영하게 됩니다.

### Q2: 왜 공유 모델이 중요한가요?
**A:** 공유 모델이 없다면 개발자는 도메인 전문가의 요구사항을 자신만의 방식으로 해석해야 합니다. 이는 API 설계에서 발생하는 오해와 비슷합니다.

실제로 많은 프로젝트에서 이런 일이 발생합니다:

**요구사항**: "고객이 주문을 취소할 수 있어야 한다"

**개발자의 해석 (잘못된 구현)**:
```csharp
public class OrderService
{
    public void CancelOrder(long orderId)
    {
        var order = _orderRepository.FindById(orderId);
        order.Status = "CANCELLED";  // 단순히 상태만 변경
        _orderRepository.Save(order);
    }
}
```

**도메인 전문가의 실제 의도 (올바른 구현)**:
```csharp
public class Order
{
    public void Cancel(CancellationReason reason)
    {
        if (!CanBeCancelled())
        {
            throw new OrderCannotBeCancelledException();
        }
        this.Status = OrderStatus.Cancelled;
        this.CancellationReason = reason;
        this.CancelledAt = DateTime.UtcNow;
        // 재고 복구, 환불 처리 등 비즈니스 로직
    }
}
```

공유 모델을 통해 이런 오해와 왜곡을 방지할 수 있습니다.

### Q3: 이벤트 스토밍은 어떻게 진행되나요?
**A:** 이벤트 스토밍은 팀 전체가 함께 비즈니스 프로세스를 모델링하는 협업 워크숍입니다. 

큰 벽에 종이를 붙이고, 참가자들이 각자 아는 비즈니스 이벤트를 포스트잇에 적어서 붙입니다. 예를 들어:

**주문 처리 도메인의 이벤트들**:
- `OrderReceived` (주문 접수됨)
- `PaymentProcessed` (결제 처리됨) 
- `InventoryReserved` (재고 확보됨)
- `OrderShipped` (주문 발송됨)
- `OrderDelivered` (주문 배송됨)

이 과정에서 자연스럽게 질문이 생깁니다:
- "OrderReceived 이벤트 후에 어떤 이벤트가 발생하나요?"
- "PaymentProcessed 이벤트는 언제 발생하나요?"
- "재고 부족 시에는 어떤 이벤트가 발생하나요?"

이런 질문들을 통해 팀 전체가 비즈니스 프로세스와 도메인 이벤트 간의 관계를 함께 이해하게 됩니다.

### Q4: 바운디드 컨텍스트를 올바르게 설정하는 방법은?
**A:** 바운디드 컨텍스트는 마이크로서비스 아키텍처에서 서비스를 나누는 것과 비슷합니다. 

먼저 도메인 전문가들이 어떻게 일하는지 관찰해보세요. 같은 용어를 사용하고, 같은 문제에 대해 이야기하는 사람들이 있다면 그들은 같은 컨텍스트에 속할 가능성이 높습니다.

**전자상거래 시스템의 바운디드 컨텍스트 예시**:

```csharp
// 주문 처리 컨텍스트
[BoundedContext("OrderManagement")]
public class OrderService
{
    public Order CreateOrder(CreateOrderCommand command) { ... }
    public void CancelOrder(CancelOrderCommand command) { ... }
}

// 결제 처리 컨텍스트  
[BoundedContext("PaymentProcessing")]
public class PaymentService
{
    public PaymentResult ProcessPayment(ProcessPaymentCommand command) { ... }
    public void RefundPayment(RefundPaymentCommand command) { ... }
}

// 재고 관리 컨텍스트
[BoundedContext("InventoryManagement")]
public class InventoryService
{
    public void ReserveInventory(ReserveInventoryCommand command) { ... }
    public void ReleaseInventory(ReleaseInventoryCommand command) { ... }
}
```

각 컨텍스트는 독립적인 데이터베이스와 API를 가지며, 이벤트를 통해 서로 소통합니다. 하지만 주의할 점은 컨텍스트가 너무 커지지 않도록 하는 것입니다. 모든 것을 하나의 컨텍스트에 넣으려 하면 단일 책임 원칙을 위반하게 됩니다.

### Q5: 도메인과 바운디드 컨텍스트는 같은 것인가요?
**A:** 아니요, 다릅니다. 도메인은 "문제 공간"에 속하고, 바운디드 컨텍스트는 "해결 공간"에 속합니다.

**도메인 (Problem Space)**:
도메인은 현실 세계의 지식 영역입니다. 예를 들어 "주문 처리"라는 도메인이 있다면, 이는 실제로 주문을 처리하는 방법, 규칙, 프로세스 등을 포함합니다.

**바운디드 컨텍스트 (Solution Space)**:
바운디드 컨텍스트는 이 도메인을 소프트웨어로 구현한 것입니다.

```csharp
// 문제 공간: 주문 처리 도메인
// - 주문 생성 규칙
// - 주문 상태 변경 규칙  
// - 주문 취소 정책

// 해결 공간: 주문 처리 바운디드 컨텍스트
[BoundedContext("OrderProcessing")]
public class OrderAggregate
{
    public void CreateOrder(CreateOrderCommand command)
    {
        // 도메인 규칙을 코드로 구현
        ValidateOrder(command);
        this.Status = OrderStatus.Pending;
        this.CreatedAt = DateTime.UtcNow;
    }
    
    public void CancelOrder()
    {
        if (!CanBeCancelled())
        {
            throw new OrderCannotBeCancelledException();
        }
        this.Status = OrderStatus.Cancelled;
    }
}
```

실제로는 하나의 도메인이 여러 개의 바운디드 컨텍스트로 나뉠 수도 있고, 여러 도메인이 하나의 바운디드 컨텍스트에 들어갈 수도 있습니다.

### Q6: 유비쿼터스 언어는 어떻게 만들어지나요?
**A:** 유비쿼터스 언어는 팀이 함께 만들어가는 공통 언어입니다. 이는 API 설계에서 일관된 네이밍 컨벤션을 만드는 것과 비슷합니다.

이벤트 스토밍 세션에서 "이걸 뭐라고 부를까?"라는 질문이 자주 나옵니다. 예를 들어, 주문이 완료된 상태를 "완료", "처리됨", "승인됨" 중 어떤 용어로 부를지 팀이 함께 결정합니다.

**유비쿼터스 언어의 예시**:

```csharp
// 팀이 합의한 용어: "주문 완료" = OrderCompleted
// 이 용어는 모든 곳에서 일관되게 사용됨

// 도메인 이벤트
public class OrderCompleted : DomainEvent
{
    public OrderId OrderId { get; private set; }
    public DateTime CompletedAt { get; private set; }
}

// 도메인 서비스
public class OrderService
{
    public void CompleteOrder(OrderId orderId)
    {
        // 비즈니스 로직
        var order = _orderRepository.FindById(orderId);
        order.Complete();
        
        // 이벤트 발행
        _domainEventPublisher.Publish(new OrderCompleted(orderId, DateTime.UtcNow));
    }
}

// API 엔드포인트
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    [HttpPost("{orderId}/complete")]
    public IActionResult CompleteOrder([FromRoute] OrderId orderId)
    {
        _orderService.CompleteOrder(orderId);
        return Ok();
    }
}
```

이렇게 정해진 용어는 문서, 코드, 대화에서 모두 동일하게 사용됩니다. 시간이 지나면서 새로운 개념이 발견되면 언어도 함께 발전합니다.

### Q7: 모든 도메인이 동일한 중요도를 가지나요?
**A:** 아니요, 도메인마다 중요도가 다릅니다. 마이크로서비스 아키텍처에서 서비스의 중요도를 분류하는 것과 비슷합니다.

**핵심 도메인 (Core Domain)**:
회사의 경쟁력과 직결되는 부분입니다. 예를 들어 전자상거래 플랫폼이라면 "주문 처리"가 핵심 도메인이 될 수 있습니다.

```csharp
// 핵심 도메인: 주문 처리 시스템
[BoundedContext("OrderProcessing")]
public class OrderAggregate
{
    // 복잡한 비즈니스 로직
    public void ProcessOrder(OrderCommand command)
    {
        // 회사만의 고유한 주문 처리 로직
        ValidateBusinessRules(command);
        CalculatePricing(command);
        ApplyDiscounts(command);
    }
}
```

**지원 도메인 (Supporting Domain)**:
필요하지만 핵심은 아닌 부분입니다. "고객 관리" 같은 기능이 여기에 해당할 수 있습니다.

```csharp
// 지원 도메인: 고객 관리 시스템
[BoundedContext("CustomerManagement")]
public class CustomerService
{
    public Customer CreateCustomer(CreateCustomerCommand command)
    {
        // 표준적인 CRUD 작업
        return _customerRepository.Save(new Customer(command));
    }
}
```

**범용 도메인 (Generic Domain)**:
회사에만 특별한 것이 아닌 일반적인 기능입니다. "이메일 발송" 같은 기능은 외부 서비스를 사용해도 됩니다.

```csharp
// 범용 도메인: 이메일 발송 (외부 서비스 사용)
[BoundedContext("Notification")]
public class EmailService
{
    public void SendEmail(EmailCommand command)
    {
        // 외부 이메일 서비스 API 호출
        _externalEmailProvider.Send(command);
    }
}
```

### Q8: DDD는 모든 소프트웨어 개발에 적합한가요?
**A:** 아니요, DDD는 특정 상황에서 특히 유용합니다.

**DDD가 효과적인 경우**:
비즈니스 로직이 복잡하고, 도메인 전문가와 개발자가 함께 협업해야 하는 프로젝트입니다.

```csharp
// 은행 시스템 예시 - 복잡한 비즈니스 규칙
[BoundedContext("Banking")]
public class AccountService
{
    public void TransferMoney(TransferCommand command)
    {
        // 복잡한 은행 규칙들
        ValidateTransferRules(command);
        CheckRegulatoryCompliance(command);
        CalculateFees(command);
        ExecuteTransfer(command);
    }
}

// 전자상거래 시스템 예시 - 복잡한 주문 처리
[BoundedContext("ECommerce")]
public class OrderService
{
    public void ProcessOrder(OrderCommand command)
    {
        // 복잡한 비즈니스 로직
        ValidateInventory(command);
        CalculatePricing(command);
        ApplyPromotions(command);
        HandlePayment(command);
    }
}
```

**DDD가 적합하지 않은 경우**:
시스템 소프트웨어나 게임 개발 같은 경우에는 DDD보다는 다른 접근법이 더 적합할 수 있습니다.

```csharp
// 게임 엔진 예시 - 기술적 복잡성이 주된 관심사
public class GameEngine
{
    public void RenderFrame()
    {
        // 렌더링 파이프라인
        UpdatePhysics();
        ProcessInput();
        RenderGraphics();
    }
}

// 운영체제 커널 예시 - 하드웨어 추상화가 주된 관심사
public class Kernel
{
    public void HandleInterrupt(InterruptRequest request)
    {
        // 하드웨어 인터럽트 처리
        SaveContext();
        DispatchInterrupt(request);
        RestoreContext();
    }
}
```

이런 분야에서는 기술적 복잡성이 비즈니스 복잡성보다 더 중요한 요소이기 때문입니다.

### Q9: 도메인 이벤트와 명령의 차이점은 무엇인가요?
**A:** 도메인 이벤트와 명령은 시간의 흐름에서 서로 다른 위치에 있습니다.

**명령 (Command)**:
"앞으로 일어날 일"에 대한 요청입니다. 이는 미래에 실행될 작업을 지시하는 것입니다.

```csharp
// 명령 예시
public class CreateOrderCommand
{
    public CustomerId CustomerId { get; set; }
    public List<OrderItem> Items { get; set; }
    public Money TotalAmount { get; set; }
}

public class ProcessPaymentCommand
{
    public OrderId OrderId { get; set; }
    public PaymentMethod PaymentMethod { get; set; }
    public Money Amount { get; set; }
}
```

**도메인 이벤트 (Domain Event)**:
"이미 일어난 일"에 대한 기록입니다. 이는 과거에 발생한 사실을 기록하는 것입니다.

```csharp
// 도메인 이벤트 예시
public class OrderCreated : DomainEvent
{
    public OrderId OrderId { get; private set; }
    public CustomerId CustomerId { get; private set; }
    public DateTime OccurredAt { get; private set; }
}

public class PaymentProcessed : DomainEvent
{
    public OrderId OrderId { get; private set; }
    public Money Amount { get; private set; }
    public DateTime OccurredAt { get; private set; }
}
```

**명령과 이벤트의 관계**:
명령이 성공적으로 실행되면 그 결과로 도메인 이벤트가 발생합니다.

```csharp
public class OrderService
{
    public void CreateOrder(CreateOrderCommand command)
    {
        // 명령 실행
        var order = new Order(command);
        _orderRepository.Save(order);
        
        // 이벤트 발행
        _domainEventPublisher.Publish(new OrderCreated(order.Id, command.CustomerId, DateTime.UtcNow));
    }
}
```

### Q10: 컨텍스트 맵의 목적은 무엇인가요?
**A:** 컨텍스트 맵은 마이크로서비스 아키텍처의 서비스 간 관계도를 그리는 것과 비슷합니다.

컨텍스트 맵은 각 바운디드 컨텍스트의 내부 구조나 세부 기능을 보여주지 않습니다. 대신 컨텍스트들 간의 관계와 데이터 흐름만 보여줍니다.

**컨텍스트 맵 예시**:

```
┌─────────────────┐     ┌─────────────────┐    ┌─────────────────┐
│   Order         │     │   Payment       │    │   Inventory     │
│   Management    │───> │   Processing    │    │   Management    │
│                 │     │                 │    │                 │
└─────────────────┘     └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐     ┌─────────────────┐    ┌─────────────────┐
│   Shipping      │     │   Notification  │    │   Customer      │
│   Management    │     │   Service       │    │   Management    │
│                 │     │                 │    │                 │
└─────────────────┘     └─────────────────┘    └─────────────────┘
```

**각 컨텍스트 간의 상호작용**:

```csharp
// Order Management → Payment Processing
public class OrderService
{
    public void ProcessOrder(OrderCommand command)
    {
        var order = CreateOrder(command);
        
        // 결제 처리 컨텍스트로 명령 전송
        _paymentService.ProcessPayment(new ProcessPaymentCommand(order.Id, command.PaymentMethod));
    }
}

// Payment Processing → Notification Service
public class PaymentService
{
    public void ProcessPayment(ProcessPaymentCommand command)
    {
        var result = ExecutePayment(command);
        
        // 알림 서비스로 이벤트 발행
        _eventPublisher.Publish(new PaymentProcessed(command.OrderId, result.Amount));
    }
}
```

이를 통해 "어떤 컨텍스트가 어떤 컨텍스트와 소통하는지", "데이터가 어떻게 흘러가는지"를 이해할 수 있습니다.

### Q11: 워크플로, 시나리오, 유스케이스, 프로세스의 차이점은 무엇인가요?
**A:** 이 용어들은 비즈니스 활동을 설명하는 데 사용되지만, 각각 다른 관점과 범위를 가집니다.

**시나리오 (Scenario)**:
고객(또는 다른 사용자)이 달성하고자 하는 목표를 설명하는 것으로, 사용자 중심의 개념입니다.

```csharp
// 시나리오: "고객이 온라인으로 주문을 하고 싶어한다"
public class PlaceOrderScenario
{
    // 사용자 관점에서의 목표
    public void CustomerWantsToPlaceOrder()
    {
        // 고객이 원하는 것: 쉽고 빠른 주문
    }
}
```

**유스케이스 (Use Case)**:
시나리오를 좀 더 구체화한 것으로, 사용자가 목표를 달성하기 위해 거쳐야 하는 상호작용과 단계를 전반적으로 서술합니다.

```csharp
// 유스케이스: 주문하기
public class PlaceOrderUseCase
{
    public void Execute(PlaceOrderRequest request)
    {
        // 1. 상품 선택
        // 2. 수량 입력
        // 3. 배송지 입력
        // 4. 결제 방법 선택
        // 5. 주문 확인
        // 6. 주문 완료
    }
}
```

**비즈니스 프로세스 (Business Process)**:
개별 사용자가 아닌 비즈니스 전체가 달성하고자 하는 목표를 설명합니다. 비즈니스 중심의 시각을 가집니다.

```csharp
// 비즈니스 프로세스: 주문 처리
public class OrderProcessingBusinessProcess
{
    public void ProcessOrder(Order order)
    {
        // 비즈니스 목표: 수익 창출, 고객 만족, 재고 관리
        ValidateOrder(order);
        ReserveInventory(order);
        ProcessPayment(order);
        ScheduleShipping(order);
    }
}
```

**워크플로 (Workflow)**:
비즈니스 프로세스의 일부를 구체적으로 설명한 것으로, 직원(또는 소프트웨어 구성 요소)이 수행해야 하는 구체적인 단계들을 나열합니다.

```csharp
// 워크플로: 주문 접수 워크플로
public class OrderReceiptWorkflow
{
    public void Execute(OrderReceiptCommand command)
    {
        // 구체적인 단계들
        Step1_ValidateOrderData(command);
        Step2_CheckInventory(command);
        Step3_CalculatePricing(command);
        Step4_CreateOrder(command);
        Step5_SendConfirmation(command);
    }
    
    private void Step1_ValidateOrderData(OrderReceiptCommand command) { ... }
    private void Step2_CheckInventory(OrderReceiptCommand command) { ... }
    private void Step3_CalculatePricing(OrderReceiptCommand command) { ... }
    private void Step4_CreateOrder(OrderReceiptCommand command) { ... }
    private void Step5_SendConfirmation(OrderReceiptCommand command) { ... }
}
```

**관계 정리**:
- **시나리오** → **유스케이스** → **비즈니스 프로세스** → **워크플로** 순으로 구체화됩니다
- 시나리오와 유스케이스는 사용자 중심, 비즈니스 프로세스와 워크플로는 비즈니스 중심입니다
- 워크플로는 한 사람 또는 한 팀이 수행할 수 있는 범위로 한정됩니다

### Q12: 용어와 코드의 관계는 어떻게 되나요?
**A:** Q11에서 설명한 각 용어들이 실제 코드에서 어떻게 구현되는지 살펴보겠습니다.

**Q11의 예시 코드를 기반으로 한 실제 구현**:

```csharp
// 1. 시나리오 → 도메인 이벤트로 표현
// Q11: "고객이 온라인으로 주문을 하고 싶어한다" → 실제로 주문이 완료된 결과
public class OrderCompleted : DomainEvent
{
    public OrderId OrderId { get; private set; }
    public CustomerId CustomerId { get; private set; }
    public DateTime CompletedAt { get; private set; }
    
    // 시나리오의 결과를 이벤트로 기록
    public OrderCompleted(OrderId orderId, CustomerId customerId, DateTime completedAt)
    {
        OrderId = orderId;
        CustomerId = customerId;
        CompletedAt = completedAt;
    }
}

// 2. 유스케이스 → 애플리케이션 서비스로 구현
// Q11: PlaceOrderUseCase의 Execute 메서드를 실제로 구현
public class PlaceOrderUseCase
{
    private readonly IOrderRepository _orderRepository;
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentService _paymentService;
    private readonly IEventPublisher _eventPublisher;
    
    public async Task<OrderResult> Execute(PlaceOrderRequest request)
    {
        // Q11에서 설명한 유스케이스 단계들을 실제로 구현
        // 1. 상품 선택 (이미 request에 포함됨)
        var products = await _inventoryService.ValidateProducts(request.ProductIds);
        
        // 2. 수량 입력 (이미 request에 포함됨)
        // 3. 배송지 입력 (이미 request에 포함됨)
        // 4. 결제 방법 선택 (이미 request에 포함됨)
        
        // 5. 주문 확인 - 실제 비즈니스 로직
        var order = new Order(request.CustomerId, products, request.ShippingAddress);
        
        // 6. 주문 완료 - 결제 처리 및 저장
        var paymentResult = await _paymentService.ProcessPayment(order.TotalAmount, request.PaymentMethod);
        await _orderRepository.Save(order);
        
        // 시나리오 결과를 이벤트로 발행
        await _eventPublisher.PublishAsync(new OrderCompleted(order.Id, order.CustomerId, DateTime.UtcNow));
        
        return new OrderResult(order.Id, paymentResult.TransactionId);
    }
}

// 3. 비즈니스 프로세스 → 도메인 서비스로 구현
// Q11: OrderProcessingBusinessProcess의 ProcessOrder 메서드를 실제로 구현
public class OrderProcessingBusinessProcess
{
    private readonly IRevenueCalculator _revenueCalculator;
    private readonly IDiscountService _discountService;
    private readonly IInventoryManager _inventoryManager;
    private readonly IShippingOptimizer _shippingOptimizer;
    
    public async Task ProcessOrder(Order order)
    {
        // Q11에서 설명한 비즈니스 목표들을 실제로 구현
        // 목표 1: 수익 창출
        var revenue = await _revenueCalculator.CalculateRevenue(order);
        order.SetRevenue(revenue);
        
        // 목표 2: 고객 만족
        var discount = await _discountService.CalculateCustomerDiscount(order);
        order.ApplyDiscount(discount);
        
        // 목표 3: 재고 관리
        await _inventoryManager.UpdateInventoryLevels(order);
        
        // 목표 4: 배송 최적화
        var optimizedRoute = await _shippingOptimizer.OptimizeShippingRoute(order);
        order.SetShippingRoute(optimizedRoute);
    }
}

// 4. 워크플로 → 구체적인 단계별 메서드로 구현
// Q11: OrderReceiptWorkflow의 Execute 메서드를 실제로 구현
public class OrderReceiptWorkflow
{
    private readonly IOrderValidator _orderValidator;
    private readonly IInventoryService _inventoryService;
    private readonly IPricingCalculator _pricingCalculator;
    private readonly IOrderFactory _orderFactory;
    private readonly INotificationService _notificationService;
    private readonly IOrderRepository _orderRepository;
    
    public async Task<WorkflowResult> Execute(OrderReceiptCommand command)
    {
        try
        {
            // Q11에서 설명한 구체적인 단계들을 실제로 구현
            await Step1_ValidateOrderData(command);
            await Step2_CheckInventory(command);
            await Step3_CalculatePricing(command);
            await Step4_CreateOrder(command);
            await Step5_SendConfirmation(command);
            
            return WorkflowResult.Success();
        }
        catch (Exception ex)
        {
            await HandleWorkflowError(command, ex);
            return WorkflowResult.Failure(ex.Message);
        }
    }
    
    // Q11의 Step1_ValidateOrderData를 실제로 구현
    private async Task Step1_ValidateOrderData(OrderReceiptCommand command)
    {
        var validationResult = await _orderValidator.Validate(command);
        if (!validationResult.IsValid)
        {
            throw new InvalidOrderException($"주문 데이터 검증 실패: {validationResult.ErrorMessage}");
        }
    }
    
    // Q11의 Step2_CheckInventory를 실제로 구현
    private async Task Step2_CheckInventory(OrderReceiptCommand command)
    {
        foreach (var item in command.Items)
        {
            var availableStock = await _inventoryService.GetAvailableStock(item.ProductId);
            if (availableStock < item.Quantity)
            {
                throw new InsufficientStockException($"상품 {item.ProductId}의 재고가 부족합니다. 요청: {item.Quantity}, 가용: {availableStock}");
            }
        }
    }
    
    // Q11의 Step3_CalculatePricing을 실제로 구현
    private async Task Step3_CalculatePricing(OrderReceiptCommand command)
    {
        var pricing = await _pricingCalculator.CalculatePricing(command.Items, command.CustomerId);
        command.SetCalculatedPricing(pricing);
    }
    
    // Q11의 Step4_CreateOrder를 실제로 구현
    private async Task Step4_CreateOrder(OrderReceiptCommand command)
    {
        var order = await _orderFactory.CreateOrder(command);
        await _orderRepository.Save(order);
        command.SetCreatedOrder(order);
    }
    
    // Q11의 Step5_SendConfirmation을 실제로 구현
    private async Task Step5_SendConfirmation(OrderReceiptCommand command)
    {
        var order = command.GetCreatedOrder();
        await _notificationService.SendOrderConfirmation(order);
    }
    
    private async Task HandleWorkflowError(OrderReceiptCommand command, Exception ex)
    {
        // 오류 발생 시 롤백 및 알림 처리
        await _notificationService.SendErrorNotification(command.CustomerId, ex.Message);
    }
}
```

**Q11과 Q12의 연결 관계**:

| Q11 용어 | Q12 코드 구현 | 책임 | Q11에서 Q12로의 발전 |
|----------|---------------|------|---------------------|
| **시나리오** | 도메인 이벤트 | "무엇이 일어났는가" 기록 | `PlaceOrderScenario` → `OrderCompleted` 이벤트 |
| **유스케이스** | 애플리케이션 서비스 | 사용자 요청 처리 | `PlaceOrderUseCase` → 실제 6단계 구현 |
| **비즈니스 프로세스** | 도메인 서비스 | 비즈니스 목표 달성 | `OrderProcessingBusinessProcess` → 4개 목표 구현 |
| **워크플로** | 구체적인 단계별 메서드 | 실행 가능한 단계들 | `OrderReceiptWorkflow` → Step1~5 실제 구현 |

**Q11에서 Q12로의 실제 구현 흐름**:

```csharp
// 1. Q11의 시나리오가 Q12에서 이벤트로 발행됨
public class OrderController : ControllerBase
{
    [HttpPost("orders")]
    public async Task<IActionResult> PlaceOrder([FromBody] PlaceOrderRequest request)
    {
        // Q11의 유스케이스가 Q12에서 실제로 실행됨
        var result = await _placeOrderUseCase.Execute(request);
        return Ok(result);
    }
}

// 2. Q11의 비즈니스 프로세스가 Q12에서 이벤트 핸들러로 실행됨
public class OrderCompletedEventHandler : IEventHandler<OrderCompleted>
{
    public async Task Handle(OrderCompleted @event)
    {
        var order = await _orderRepository.GetById(@event.OrderId);
        
        // Q11의 비즈니스 프로세스가 Q12에서 실제로 실행됨
        await _orderProcessingBusinessProcess.ProcessOrder(order);
    }
}

// 3. Q11의 워크플로가 Q12에서 워크플로 엔진으로 실행됨
public class WorkflowEngine
{
    public async Task ExecuteWorkflow(string workflowName, object command)
    {
        switch (workflowName)
        {
            case "OrderReceipt":
                // Q11의 워크플로가 Q12에서 실제로 실행됨
                var workflow = new OrderReceiptWorkflow();
                await workflow.Execute((OrderReceiptCommand)command);
                break;
        }
    }
}
```

**Q11과 Q12의 관계 요약**:
- **Q11**: 용어의 개념적 정의와 간단한 코드 구조
- **Q12**: Q11의 개념을 실제 프로덕션 코드로 구현
- **연결점**: Q11의 각 용어가 Q12에서 구체적인 의존성 주입, 오류 처리, 비동기 처리 등을 포함한 완전한 구현으로 발전

## 핵심 메시지

1. **개발자의 본질적 역할은 코드 작성이 아니라 소프트웨어를 통해 문제를 해결하는 것**
2. **공유 모델을 통해 번역 과정에서 발생하는 왜곡을 방지**
3. **비즈니스 이벤트 중심으로 도메인을 이해하고 모델링**
4. **복잡한 도메인을 하위 도메인으로 분할하여 관리**
5. **바운디드 컨텍스트를 통해 명확한 경계와 자율성 확보**
6. **유비쿼터스 언어로 팀 전체의 공유된 이해 구축**
