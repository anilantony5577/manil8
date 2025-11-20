#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define MAX_FRAMES 100

int total_frames;      // Total number of frames to send
int window_size;       // Window size

// Simulate sending a frame with 90% success rate
int send_frame(int frame) {
    return rand() % 10 < 9;   // 90% success, 10% failure
}

// Simulate receiving an acknowledgment (always successful here)
int receive_ack(int frame) {
    return 1;
}

// Go-Back-N ARQ Protocol Simulation
void go_back_n() {
    int base = 0;           // First frame in the window
    int next_frame = 0;     // Next frame to send

    while (base < total_frames) {
        // Send all frames in the window
        for (int i = 0; i < window_size && next_frame < total_frames; i++) {
            printf("Sending Frame %d\n", next_frame);

            if (!send_frame(next_frame)) {
                printf("Frame %d lost during transmission.\n", next_frame);
                break;  // Go-Back-N: stop sending and wait for timeout
            }

            next_frame++;
        }

        // Check acknowledgments
        int all_acknowledged = 1;

        for (int i = base; i < next_frame; i++) {
            if (send_frame(i)) {
                printf("ACK received for Frame %d\n", i);
                base++;
            } else {
                printf("ACK lost for Frame %d. Go-Back-N from Frame %d\n", i, i);
                next_frame = base;       // Resend from base
                all_acknowledged = 0;
                break;
            }
        }

        if (!all_acknowledged) {
            printf("Resending window from Frame %d\n", base);
        }
    }

    printf("All frames sent and acknowledged successfully.\n");
}

int main() {
    srand(time(NULL));  // Seed random number generator

    printf("Enter total number of frames to send: ");
    scanf("%d", &total_frames);

    if (total_frames <= 0 || total_frames > MAX_FRAMES) {
        printf("Invalid number of frames. Enter between 1 and %d.\n", MAX_FRAMES);
        return 1;
    }

    printf("Enter window size: ");
    scanf("%d", &window_size);

    if (window_size <= 0 || window_size > total_frames) {
        printf("Invalid window size. Enter between 1 and %d.\n", total_frames);
        return 1;
    }

    go_back_n();   // Run simulation
    return 0;
}
