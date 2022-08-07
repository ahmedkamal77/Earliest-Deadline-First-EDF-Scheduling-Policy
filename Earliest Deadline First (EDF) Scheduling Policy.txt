// Earliest Deadline First (EDF) Scheduling Policy.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>
using namespace std;

struct task
{
    double r;
    double p;
    double e;
    double d;
    double executed;
    double T_start;
    double T_end;
    int x;
    int y;
    double now;
};

bool r_comp(task& a, task& b)
{
    return a.d < b.d;
}

bool r_comp2(task& a, task& b)
{
    return a.r < b.r;
}

int main()
{
    double b;

    bool f = false;
    double round = 0; //round refers to the time of each round
    int n, j = 7; // j refers to number of jobs for all tasks
    double current = 0;  //current refers to current moment

    cout << "Enter number of tasks: ";
    cin >> n;
    vector<task> tasks(n);
    vector<task>::iterator it = tasks.begin();
    cout << "Enter round period: ";
    cin >> round;
    cout << "Enter Tasks info as Period, Execution e.g: 0 6" << endl;
    cout << "                                           1 8" << endl;
    cout << "                                           ..." << endl;
    for (int i = 0; i < n; ++i)
    {
        cin >> tasks[i].p;
        cin >> tasks[i].e;
        tasks[i].r = 0;
        tasks[i].d = tasks[i].p;
        tasks[i].x = i + 1;
        tasks[i].y = 1;
        tasks[i].now = 0;
        tasks[i].T_start = tasks[i].r;
        tasks[i].T_end = 0;
        tasks[i].executed = 0;
    }
    sort(tasks.begin(), tasks.end(), r_comp);
    deque <task> execution;
    task t;

    while (j--)
    {
        for (int i = 0; i < tasks.size(); i++)
            if (tasks[i].d == tasks[0].d && tasks[i].r <= current)
            {
                execution.push_back(tasks[i]);
                it = tasks.begin();
                for (int j = 0; j < i; j++)
                    it++;
                tasks.erase(it);
            }

        if (execution.size() == 0)
        {
            sort(tasks.begin(), tasks.end(), r_comp2);
            execution.push_back(tasks[0]);
            it = tasks.begin();
            tasks.erase(it);
        }

        t = execution.front();
            
        while (t.executed != t.e)
        {
            t = execution.front();
            if (t.executed == 0)
                t.T_start = current;
           
            current += round;
            t.executed += round;
            t.T_end = current;

            //check ready tasks for execution
            it = tasks.begin();
            for (int i = 0; i < tasks.size(); i++)
            {
                if (tasks[i].d == t.d && tasks[i].r <= current)    //tasks with the same deadline will be executed as round-robin
                {
                    execution.push_back(tasks[i]);
                    it = tasks.begin();
                    for (int j = 0; j < i; j++)
                         it++;
                    tasks.erase(it);
                }

                else if (tasks[i].d < t.d && tasks[i].r <= current)    //tasks with the different deadlines will be executed as Earliest Deadline First
                {
                    //swap between the two tasks
                    task f = tasks[i]; 
                    tasks[i] = t;
                    t = f;
                    execution.pop_front();
                    execution.push_back(t);
                }
            }

            //cout << ": j" << t.x << t.y << endl; 
            if (current > (t.d))
            {
                cout << "j" << t.x << t.y << " has Deadline Broken" << endl;
                f = true;
                break;
            }

            //transfer running task to the end of the execution queue
            execution.pop_front();
            execution.push_back(t);
            
        }

        if (f)
            break;
        else
        {
            cout << "from T" << t.T_start << " to T" << t.T_end << ": j" << t.x << t.y << endl;
            //Preparing the next job of the task
            t.y++;
            t.r += t.p;
            t.d += t.p;
            t.now = t.r;
            t.T_start = t.r;
            t.T_end = 0;
            t.executed = 0;
            b = t.d;
            execution.pop_back();
            tasks.insert(tasks.begin(), t);
            sort(tasks.begin(), tasks.end(), r_comp);
        }
    }

    return 0;

}

