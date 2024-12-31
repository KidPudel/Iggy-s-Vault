# Strategy Pattern
---
Behavioral pattern where you define a family strategies and can change them at runtime
```go
type PaymentStrategy interface {
    Pay(amount float64) error
}

type CreditCardPayment struct {}
func (c *CreditCardPayment) Pay(amount float64) error {
    // Logic for credit card payment
    return nil
}

type PaymentUsecase struct {
    strategy PaymentStrategy
}

func NewPaymentUsecase(strategy PaymentStrategy) *PaymentUsecase {
    return &PaymentUsecase{strategy: strategy}
}

// change at runtime
func (u *PaymentUsecase) ChangePayment(strategy PaymentStrategy) {
	u.strategy = strategy
}

func (u *PaymentUsecase) ExecutePayment(amount float64) error {
    return u.strategy.Pay(amount)
}

```

# Factory Pattern
---
Creation pattern for creating object without specifying the exact type
```go
type OCRFactory struct {}

type OCR interface {
	ReadText(file []byte) string
}

func (factory OCRFactory) CreateOCR(ocrKind string) (OCR, error) {
	switch ocrKind {
		case "tesseract":
			return NewTesseract(), nil
		case "google":
			return NewGoogle(), nil
		default:
			return nil, errors.New("unknown OCR kind")
	}
}
```