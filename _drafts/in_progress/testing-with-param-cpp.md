class Fixture : public ::testing::TestWithParam<int> {
    //Random initialisation
};

TEST_P(Fix, Test1){}

INSTANTIATE_TEST_CASE_P(Instantiation, Fixture, ::testing::Range(1, 10));