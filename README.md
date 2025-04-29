#include <iostream>
#include <vector>
#include <random>
#include <algorithm>
#include <chrono>

constexpr int NUM_ITEMS = 10000;
constexpr int NUM_WORKERS = 10;

// Random number generators
std::mt19937 rng(std::chrono::steady_clock::now().time_since_epoch().count());

// Uniformly distributed time between 100 and 300
double get_assembly_time() {
    std::uniform_real_distribution<double> dist(100.0, 300.0);
    return dist(rng);
}

// Normally distributed polishing time with mean 20 and stddev 7 (truncated below 5)
double get_polishing_time() {
    std::normal_distribution<double> dist(20.0, 7.0);
    double time;
    do {
        time = dist(rng);
    } while (time < 5.0);
    return time;
}

// Find the earliest available polishing machine
int find_next_available_machine(const std::vector<double>& machine_ready_times) {
    return std::min_element(machine_ready_times.begin(), machine_ready_times.end()) - machine_ready_times.begin();
}

double simulate(int num_machines) {
    std::vector<double> worker_time(NUM_WORKERS, 0.0);
    std::vector<double> machine_ready(num_machines, 0.0);

    double total_wait_time = 0.0;

    for (int i = 0; i < NUM_ITEMS; ++i) {
        int worker_id = i % NUM_WORKERS;

        // Assembly
        double assembly_time = get_assembly_time();
        double assembly_done = worker_time[worker_id] + assembly_time;

        // Find polishing machine
        int machine_id = find_next_available_machine(machine_ready);
        double wait_time = std::max(0.0, machine_ready[machine_id] - assembly_done);
        double polishing_start = assembly_done + wait_time;
        double polishing_time = get_polishing_time();
        double polishing_done = polishing_start + polishing_time;

        // Update times
        machine_ready[machine_id] = polishing_done;
        worker_time[worker_id] = polishing_done;

        total_wait_time += wait_time;
    }

    return total_wait_time / NUM_ITEMS;
}

int main() {
    std::cout << "Average wait time with 1 polishing machine: " << simulate(1) << " seconds\n";
    std::cout << "Average wait time with 2 polishing machines: " << simulate(2) << " seconds\n";
    std::cout << "Average wait time with 3 polishing machines: " << simulate(3) << " seconds\n";

    return 0;
}
