<template>
    <div class="dashboard">
        <div
            class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3"
        >
            <div class="col-md order-md-first text-center text-md-left">
                <h2>
                    <span class="small text-uppercase text-muted d-block">Statistics</span>
                    <span
                        v-if="this.filter.dateStart && this.filter.dateEnd"
                    >{{formatDateEnUs(this.filter.dateStart)}} - {{formatDateEnUs(this.filter.dateEnd)}}</span>
                </h2>
            </div>

            <div class="btn-toolbar mb-2 mb-md-0">
                <div class="btn-group mr-2">
                    <kendo-combobox
                        v-model="selectedUserIdStr"
                        :data-source="users"
                        :data-text-field="'fullName'"
                        :data-value-field="'id'"
                        :placeholder="'Select assignee...'"
                        @open="userFilterOpen"
                        @change="userFilterValueChange"
                        style="width: 250px;"
                        :template="userFilterItemTemplate"
                    ></kendo-combobox>
                    <kendo-buttongroup>
                        <button
                            data-icon="calendar"
                            class="k-flat"
                            @click="(e) => onMonthRangeTap(3)"
                        >3 Months</button>
                        <button
                            data-icon="calendar"
                            class="k-flat"
                            @click="(e) => onMonthRangeTap(6)"
                        >6 Months</button>
                        <button
                            data-icon="calendar"
                            class="k-flat"
                            @click="(e) => onMonthRangeTap(12)"
                        >1 Year</button>
                    </kendo-buttongroup>
                </div>
            </div>
        </div>

        <div class="card">
            <h3 class="card-header">Active Issues</h3>
            <div class="card-block">
                <ActiveIssues :statusCounts="statusCounts" />

                <div class="row">
                    <div class="col-sm-12">
                        <h3>All issues</h3>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { Observable } from 'rxjs';
import { Store } from '@/core/state/app-store';
import { PtUserService } from '@/core/services/pt-user-service';
import { PtUser } from '@/core/models/domain/pt-user.model';
import { EMPTY_STRING } from '@/core/helpers/string-helpers';

import {
    DashboardRepository,
    DashboardFilter,
    FilteredIssues,
} from '@/repositories/dashboard-repository';
import { DashboardService } from '@/services/dashboard-service';
import {
    TypeCounts,
    PriorityCounts,
    StatusCounts,
} from '@/shared/models/ui/stats';
import { formatDateEnUs } from '@/core/helpers/date-utils';
import ActiveIssues from '@/components/dashboard/ActiveIssues.vue';

interface DateRange {
    dateStart: Date;
    dateEnd: Date;
}

@Component({
    components: {
        ActiveIssues,
    },
})
export default class DashboardPage extends Vue {
    public filter: DashboardFilter = {};
    public statusCounts: StatusCounts = {
        activeItemsCount: 0,
        closeRate: 0,
        closedItemsCount: 0,
        openItemsCount: 0,
    };
    public categories: Date[] = [];
    public itemsOpenByMonth: number[] = [];
    public itemsClosedByMonth: number[] = [];

    private store: Store = new Store();
    private dashboardRepo: DashboardRepository = new DashboardRepository();
    private dashboardService: DashboardService = new DashboardService(
        this.dashboardRepo
    );
    private userService: PtUserService = new PtUserService(this.store);
    private users: PtUser[] = [];
    private users$: Observable<PtUser[]> = this.store.select<PtUser[]>('users');
    private selectedUserIdStr: string = EMPTY_STRING;

    public created() {
        this.users$.subscribe(users => {
            this.users = users;
        });
        this.refresh();
    }
    /*
    private userFilterItemTemplate = `
        <div class="row" style="margin-left: 5px;">
            <img class="li-avatar rounded mx-auto d-block" src=#=avatar# />
            <span style="margin-left: 5px;">#= fullName #</span>
        </div>
    `;
    */

    private userFilterItemTemplate(u: PtUser) {
        return `
        <div class="row" style="margin-left: 5px;">
            <img class="li-avatar rounded mx-auto d-block" src=${u.avatar} />
            <span style="margin-left: 5px;">${u.fullName}</span>
        </div>

        `;
    }

    private refresh() {
        this.dashboardService.getStatusCounts(this.filter).then(result => {
            this.statusCounts = result;
        });
    }

    private userFilterOpen() {
        this.userService.fetchUsers();
    }

    private userFilterValueChange(e: any) {
        // tslint:disable-next-line:no-console
        console.log('Filter on userId', this.selectedUserIdStr);

        if (this.selectedUserIdStr) {
            // tslint:disable-next-line:no-console
            console.log('Filter on userId', this.selectedUserIdStr);
            this.filter.userId = Number(this.selectedUserIdStr);
        } else {
            // tslint:disable-next-line:no-console
            console.log('Clear Filter on userId');
            this.filter.userId = undefined;
        }
        // this.filter.userId = 1;
        this.refresh();
    }

    private onMonthRangeTap(months: number) {
        const range = this.getDateRange(months);
        this.filter = {
            userId: this.filter.userId,
            dateEnd: range.dateEnd,
            dateStart: range.dateStart,
        };
        this.refresh();
    }

    private getDateRange(months: number): DateRange {
        const now = new Date();
        const start = new Date();
        start.setMonth(start.getMonth() - months);
        return {
            dateStart: start,
            dateEnd: now,
        };
    }

    private formatDateEnUs(date: Date) {
        return formatDateEnUs(date);
    }
}
</script>
