
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#ifdef _WIN32
  #include <windows.h>
  #define SLEEP_MS(ms) Sleep(ms)
#else
  #include <unistd.h>
  #define SLEEP_MS(ms) usleep((ms) * 1000)
#endif

#define LINE_MAX_LEN 256

static int max(int a, int b) { return a > b ? a : b; }

/* Draw the DP table. Rows > upto are shown as '.' (not computed yet). */
static void show(int **dp, int n, int W, const int *wt, const int *val,
                 int upto, int cur, int delay_ms)
{
    if (delay_ms > 0) printf("\033[2J\033[H");
    printf("        cap ");
    for (int c = 0; c <= W; c++) printf("%4d", c);
    printf("\n        ----");
    for (int c = 0; c <= W; c++) printf("----");
    printf("\n");
    for (int i = 0; i <= n; i++) {
        char label[24];
        if (i == 0) snprintf(label, sizeof label, "none");
        else        snprintf(label, sizeof label, "w%d,v%d", wt[i-1], val[i-1]);
        printf("%s%-8s|", i == cur ? "->" : "  ", label);
        for (int c = 0; c <= W; c++) {
            if (i <= upto) printf("%4d", dp[i][c]);
            else           printf("   .");
        }
        printf("\n");
    }
    printf("\n");
    fflush(stdout);
    if (delay_ms > 0) SLEEP_MS(delay_ms);
}

/* Fills the table, returns best value. */
static int knapsack(int n, int W, const int *wt, const int *val, int delay_ms)
{
    int **dp = malloc((n + 1) * sizeof *dp);
    for (int i = 0; i <= n; i++) dp[i] = calloc(W + 1, sizeof **dp);

    show(dp, n, W, wt, val, 0, 0, delay_ms);
    for (int i = 1; i <= n; i++) {
        for (int c = 0; c <= W; c++) {
            dp[i][c] = dp[i-1][c];                       /* skip item i */
            if (wt[i-1] <= c)                            /* take item i */
                dp[i][c] = max(dp[i][c], dp[i-1][c - wt[i-1]] + val[i-1]);
        }
        show(dp, n, W, wt, val, i, i, delay_ms);
    }

    /* backtrack to list chosen items */
    printf("Items chosen (weight,value):");
    int c = W;
    for (int i = n; i >= 1; i--)
        if (dp[i][c] != dp[i-1][c]) {
            printf(" (%d,%d)", wt[i-1], val[i-1]);
            c -= wt[i-1];
        }
    printf("\n");

    int best = dp[n][W];
    for (int i = 0; i <= n; i++) free(dp[i]);
    free(dp);
    return best;
}

int main(int argc, char *argv[])
{
    if (argc < 2) {
        fprintf(stderr, "Usage: %s <input_file> [delay_ms]\n", argv[0]);
        return 1;
    }
    int delay_ms = argc > 2 ? atoi(argv[2]) : 0;

    FILE *in = fopen(argv[1], "r");
    if (!in) { perror("Cannot open input file"); return 1; }
    FILE *out = fopen("result.txt", "w");
    if (!out) { perror("Cannot create result.txt"); return 1; }

    char line[LINE_MAX_LEN];
    int test_no = 0, have_test = 0;

    while (1) {
        /* find next "#Test" marker */
        int found = 0;
        while (fgets(line, sizeof line, in))
            if (strncmp(line, "#Test", 5) == 0) { found = 1; break; }
        if (!found) break;
        have_test = 1;

        int n, W;
        /* header line n W (skip blank lines) */
        while (fgets(line, sizeof line, in))
            if (sscanf(line, "%d %d", &n, &W) == 2) break;

        int *wt = malloc(n * sizeof *wt), *val = malloc(n * sizeof *val);
        for (int i = 0; i < n; ) {
            if (!fgets(line, sizeof line, in)) break;
            if (sscanf(line, "%d %d", &wt[i], &val[i]) == 2) i++;
        }

        test_no++;
        printf("===== Test %d: n=%d W=%d =====\n", test_no, n, W);
        int best = knapsack(n, W, wt, val, delay_ms);
        printf("Maximum value = %d\n\n", best);
        fprintf(out, "#Test%d %d %d  %d\n", test_no, n, W, best);

        free(wt); free(val);
    }
    if (!have_test) fprintf(stderr, "No test cases found.\n");

    fclose(in); fclose(out);
    return 0;
}
