>Часто возможны ситуации, в которых необходимо производить отмену некоторых действий (например откат изменений файла), т.е. необходима возможность возврата к предыдущему состоянию.

>Паттерн Опекун (Memento) реализует идею вынесения логики сохранения состояний в отдельный объект, ответственный за сохранение состояний объекта и возвращение его в предыдущее состояние.

![[memento.png]]

>Memento - _снимок_ объекта в какой-то момент времени. Опекун отвечает за хранение этих снимков и по возможности вернуться к предыдущему состоянию объекта.

**Преимущества паттерна:**
- Позволяет снять с класса задачу о сохранении своих предыдущих состояний.
- Предоставляет возможность откатывать состояние класса на несколько состояний назад.
- Позволяет реализовать различную логику сохранения состояний.

**Недостатки паттерна:**
- Требуется управлять опекуном.
- Требуется реализация механизма очистки и определения того, нужны снимки или нет.
- Затраты памяти на сохранение снимков состояний.

>В объект необходимо добавить метод createMemento для сохранения состояния и метод restoreMemento для восстановления состояния по снимку.

>Класс опекуна должен представлять собой контейнер для снимков состояний объекта, должен позволять устанавливать состояния, получать состояние, а также производить возврат состояния. Возможны различные способы хранения состояний – в виде стека, дерева, изменений относительно предыдущего состояния.

>Класс снимка должен иметь методы set и get для сохранения и установления состояния.

```c++
# include <iostream>
# include <memory>
# include <list>

using namespace std;

class Memento;

class Caretaker
{
public:
	unique_ptr<Memento> getMemento();
	void setMemento(unique_ptr<Memento> memento);

private:
	list<unique_ptr<Memento>> mementos;
};

class Originator
{
public:
	Originator(int s) : state(s) {}
	
	const int getState() const { return state; }
	void setState(int s) { state = s; }
	
	std::unique_ptr<Memento> createMemento() { return make_unique<Memento>(*this); }
	void restoreMemento(std::unique_ptr<Memento> memento);

private:
	int state;
};

class Memento
{
	friend class Originator;

public:
	Memento(Originator o) : originator(o) {}

private:
	void setOriginator(Originator o) { originator = o; }
	Originator getOriginator() { return originator; }

private:
	Originator originator;
};

# pragma region Methods Caretaker
void Caretaker::setMemento(unique_ptr<Memento> memento)
{
	mementos.push_back(move(memento));
}

unique_ptr<Memento> Caretaker::getMemento() {
	
	unique_ptr<Memento> last = move(mementos.back());
	
	mementos.pop_back();
	
	return last;
}
# pragma endregion

# pragma region Method Originator
void Originator::restoreMemento(std::unique_ptr<Memento> memento)
{
	*this = memento->getOriginator();
}
# pragma endregion

int main()
{
	auto originator = make_unique<Originator>(1);
	auto caretaker = make_unique<Caretaker>();
	
	cout << "State = " << originator->getState() << endl;
	caretaker->setMemento(originator->createMemento());
	
	originator->setState(2);
	cout << "State = " << originator->getState() << endl;
	caretaker->setMemento(originator->createMemento());
	originator->setState(3);
	cout << "State = " << originator->getState() << endl;
	caretaker->setMemento(originator->createMemento());
	
	originator->restoreMemento(caretaker->getMemento());
	cout << "State = " << originator->getState() << endl;
	originator->restoreMemento(caretaker->getMemento());
	cout << "State = " << originator->getState() << std::endl;
	originator->restoreMemento(caretaker->getMemento());
	cout << "State = " << originator->getState() << std::endl;
}

```