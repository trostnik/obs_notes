Lambda can capture local variables of function even when function done working(because local variables getting destroyed after that). For that Kotlin uses boxing, a class that holds reference to the value
class Ref<T>(var someValue: T)